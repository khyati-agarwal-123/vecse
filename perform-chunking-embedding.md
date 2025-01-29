## Perform Chunking With Embedding {#GUID-9CB40305-392A-4CB0-AD49-7730504BEC18}

In these examples, you can see how to explore the `VECTOR_CHUNKS` SQL function along with chainable utility PL/SQL functions to split large textual extracts and documents into chunks and then represent each chunk as a vector embedding. 

To embed large textual data, you first need to prepare it in a format that can be processed by embedding models. You first transform the data into plain text, split the resulting text into smaller chunks of text, and then transform each chunk into a vector. This is done to comply with the input limits set by embedding models. Chunks can be words (to capture specific words or word pieces), sentences (to capture a specific context), or paragraphs (to capture broader themes).

  * [Convert Text to Chunks With Custom Chunking Specifications](convert-text-chunks-custom-chunking-specifications.md)  
A chunked output, especially for long and complex documents, sometimes loses contextual meaning or coherence with its parent content. In this example, you can see how to refine your chunks by applying custom chunking specifications. 
  * [Convert File to Text to Chunks to Embeddings Within Oracle Database](convert-file-text-chunks-embeddings-oracle-database.md)  
First convert a PDF file to text, split the text into chunks, and then create vector embeddings on each chunk by accessing a vector embedding model stored in the database. 
  * [Convert File to Embeddings Within Oracle Database](convert-file-embeddings-oracle-database.md)  
Directly extract vector embeddings from a PDF document, using a single-step statement, by accessing a vector embedding model stored in the database. 
  * [Generate and Use Embeddings for an End-to-End Search](generate-and-use-embeddings-end-end-search.md)  
First generate vector embeddings from textual content by using a vector embedding model stored in the database, and then populate and query a vector index. At query time, you also vectorize the query criteria on the fly. 



**Parent topic:** [Vector Generation Examples](vector-generation-examples.md)
