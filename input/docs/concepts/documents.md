Title: Documents
Order: 1
Description: A document is a combination of content and metadata and is what conveys information through the framework.
---
A *document* is a combination of *content* and *[metadata](/docs/concepts/metadata)* as it makes it's way through the system. The content of a document is what most modules manipulate and is what you will presumably output to disk at the end of the pipeline. The metadata serves as a way to pass information between modules about each bit of content. Once a value is added to the metadata by one module, it can never be removed by a subsequent one (though it can be overwritten). It's also important to note that documents are immutable. Though we often talk about documents being "transformed" or "manipulated" by modules, this isn't strictly accurate. Instead modules return a new copy of the document with different content and/or additional metadata, while maintaining all the metadata the original document had.

For example, this visualizes a single document that contains some content as well as two metadata values:

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

# Accessing Documents

During execution you can access all the documents generated by each pipeline using the `IDocumentCollection` interface, which is available via the `IExecutionContext.Documents` property for the current context. The `IExecutionContext` is available from many places including in many module constructors via a delegate and in most supported template languages such as [Razor](/modules/razor).

You can also access documents generated by a given pipeline using the [Documents](/modules/documents) module. This module inserts the documents from one pipeline into another one and can be very useful when setting up multi-pipeline generations.
