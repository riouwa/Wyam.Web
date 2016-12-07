Title: Concepts
Description: Describes the primary concepts of modules, pipelines, documents, and metadata.
Order: 20
---
The primary concepts in Wyam are *recipes*, *themes*, *modules*, *pipelines*, *documents*, and *metadata*. 

# Recipes
---

A *recipe* is a pre-configured series of modules and pipelines (see below). <a href="/recipes-themes">Each recipe</a> can be thought of as it's own special purpose static site generator. For example, the <a href="/recipes-themes/blog">Blog</a> recipe can be thought of as analogous to a tool like Jekyll. Recipes can still make use of <a href="/getting-started/configuration">configuration files</a> to further tweak the generation process, but they aren't required. 

# Themes
---

Each *theme* contains a set of content such as CSS files, layouts, etc. that apply to a specific recipe. A theme can be used to get up and running quickly with a given recipe by providing the set of resources the recipe requires. Because each recipe is a totally different set of modules, pipelines, metadataa, etc. a given theme is only applicable to a specific recipe.

# Modules
---

A *module* is a small single-purpose component that takes documents as input, does something based on those documents (possibly transforming them), and outputs documents as a result of whatever operation was performed.

# Pipelines
---

A *pipeline* is a series of modules executed in sequence that results in final output documents. A given Wyam configuration can have multiple pipelines which are executed in sequence, and subsequent pipelines have access to the documents from the previous pipelines.

Conceptually, a simple pipeline looks like:

<div class="mermaid">
    graph TD
        D1("Empty Document")
        D1-->Module1["Module 1"]
        Module1-->D2("Document A")
        Module1-->D3("Document B")
        D2-->Module2["Module 2"]
        D3-->Module2
        Module2-->D4("Document C")
        Module2-->D5("Document D")
</div>

In the visualization above, the first module may have read some files (in this case 2 files) and stuck some information about those files such as name and path in the document metadata. Then the second module may have transformed the files (for example, from Markdown to HTML).

It's not unusual for a real-world generation to contain many different pipelines. Many times this is helpful if you need to reuse the output from one of the pipelines or want to separate the different generation steps.

<div class="mermaid">
    graph TD
        subgraph Second Pipeline
            D6("Empty Document")
            D6-->Module3["Module 3"]
            Module3-->D7("Document E")
            D7-->Module4["Module 4"]
            Module4-->D8("Document F")
        end
        subgraph First Pipeline
            D1("Empty Document")
            D1-->Module1["Module 1"]
            Module1-->D2("Document A")
            Module1-->D3("Document B")
            D2-->Module2["Module 2"]
            D3-->Module2
            Module2-->D4("Document C")
            Module2-->D5("Document D")
        end
</div>

# Documents
---

A *document* is a combination of *content* and *metadata* as it makes it's way through the system. The content of a document is what most modules manipulate and is what you will presumably output at the end of the pipeline. The metadata serves as a way to pass information to and from modules to other modules (see below). Once a value is added to the metadata by one module, it can never be removed by a subsequent one (though it can be overwritten). It can be important to note that documents are immutable. Though we often talk about documents being "transformed" or "manipulated" by modules, this isn't strictly accurate. Instead modules return a new copy of the document with different content and/or additional metadata.

<div class="mermaid">
    graph TD
        subgraph Document
            subgraph Metadata
                Title
                Published
            end
            Content
        end
</div>

## Accessing Documents

During execution you can access all the documents generated by each pipeline so far via the `IDocumentCollection` interface, which is available via the `Documents` property in the `IExecutionContext` provided to module constructors and also available in most templates (for example, in [Razor templates](/modules/razor) the page property `Documents` contains the current `IDocumentCollection`).

# Metadata
---

Along with it's content, every document contains *metadata*. As with documents, metadata is immutable and you must clone a document to add additional metadata. Several modules, such as [Meta](/modules/meta), are designed to allow you to manipulate document metadata as part of your pipeline.

Metadata is the primary means of passing information between modules and pipelines. For example, when a file is [read from disk](/modules/readfiles), metadata is set that records where on disk the file came from, it's file name, and other information. When the file is later [written back to disk](/modules/writefiles), this metadata is used to determine where the file should go and what filename to use. Another example would be using metadata to define tags for your blog posts. You could create a "Tags" metadata field in the [front matter](/modules/frontmatter) of your post file and then read that metadata later to create tag clouds, lists of similar posts, etc.

## Accessing Metadata

Metadata is available via the `Metadata` property of every `IDocument`. The `IMetadata` interface implements `IReadOnlyDictionary<string, object>` for easy access. Every `IDocument` also implements `IReadOnlyDictionary<string, object>` and passes any calls through to it's metadata (so you'll rarely actually use the `Metadata` property and just access metadata directly through the document).

## Metadata Type Conversion 

All metadata is represented internally as raw objects. This allows you to store not just strings, but more complex data as well. However, when you access metadata you probably don't want to think about how it's stored or what the orignal type was. For example, [YAML](/modules/yaml) doesn't really distinguish between numbers and strings when it reads data, it's only when getting a value that you care. To make metadata as easy to work with as possible, Wyam includes a very powerful type conversion capability that lets you convert nearly any metadata value to any other compatible type.
    
When converting metadata values, all .NET type conversion techniques are checked including `TypeConverter`, `IConvertible`, casting, etc. The conversion support is provided by the [UniversalTypeConverter](http://www.codeproject.com/Articles/248440/Universal-Type-Converter) library.

If you request an `IList<T>`, `IEnumerable<T>`, or array of `T` and the metadata value is also enumerable, all elements will be converted to the requested type `T` and those that cannot be converted will be omitted from the result. If the metadata value is not enumerable, it will be returned as a single element of the requested enumerable type. 
    
## Metadata Lookup

There are several extensions to make working with documents and metadata easier. One of the more powerful ones lets you generate an `ILookup<T, IDocument>` from a sequence of documents based on a metadata key. The signature of the extension method is `ILookup<T, IDocument> ToLookup<T>(this IEnumerable<IDocument> documents, string key)` where `key` is the metadata key that you want to generate a lookup for.

For example, say you have a sequence of documents, some of which contain metadata for the key "Tags". Also, assume that some of the documents with "Tags" metadata contain a single value some contain arrays. If you simply call `Documents.ToLookup<string>("Tags")` you will get back an `ILookup<T, IDocument>` keyed by each possible tag string with a sequence of the documents that contain that tag as the value.