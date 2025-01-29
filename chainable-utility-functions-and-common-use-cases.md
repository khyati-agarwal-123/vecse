## About Chainable Utility Functions and Common Use Cases {#GUID-63773EF5-071E-4C02-9EC9-D4E0BA2CE2A2}

These are intended to be a set of chainable and flexible "stages" through which you pass your input data to transform into a different representation, including vectors.

Supplied Chainable Utility Functions

You can combine a set of chainable utility (`UTL`) functions together in an end-to-end pipeline. 

Each pipeline or **transformation chain** can include a single function or a combination of functions, which are applied to source documents as they are transformed into other representations (text, chunks, summary, or vector). These functions are chained together, such that the output from one function is used as an input for the next. 

Each chainable utility function performs a specific task of transforming data into other representations, such as converting data to text, converting text to chunks, or converting the extracted chunks to embeddings. 

At a high level, the supplied chainable utility functions include:

Function | Description | Input and Return Value  
---|---|---  
`UTL_TO_TEXT()` |  Converts data (for example, Word, HTML, or PDF documents) to plain text. |  Accepts the input as `CLOB` or `BLOB`. <br>Returns a plain text version of the document as `CLOB`.   
`UTL_TO_CHUNKS()` |  Converts data to chunks. |  Accepts the input as plain text (`CLOB` or `VARCHAR2`). <br>Splits the data to return an array of chunks (`CLOB`).   
`UTL_TO_EMBEDDING()` |  Converts data to a single embedding. |  Accepts the input as plain text (`CLOB`) or image (`BLOB`). <br>Returns a single embedding (`VECTOR`).   
`UTL_TO_EMBEDDINGS()` |  Converts an array of chunks to an array of embeddings. |  Accepts the input as an array of chunks (`VECTOR_ARRAY_T`). <br>Returns an array of embeddings (`VECTOR_ARRAY_T`).   
`UTL_TO_SUMMARY()` |  Generates a concise summary for data, such as large or complex documents. |  Accepts the input as plain text (`CLOB`). <br>Returns a summary in plain text as `CLOB`.   
`UTL_TO_GENERATE_TEXT()` |  Generates text for prompts and images. |  Accepts the input as text data (`CLOB`) for prompts, or as media data (`BLOB`) for media files such as images. <br>Processes this information to return `CLOB` containing the generated text.   
`RERANK()` |  Retrieves more relevant search output. |  Accepts the input as a query (`CLOB`) and a list of documents (`JSON`). <br>Processes this information to return a `JSON` object containing a reranked list of documents, sorted by score.   
  
Sequence of Chains

Chainable utility functions are designed to be flexible and modular. You can create transformation chains in various sequences, depending on your use case. 

For example, you can directly extract vectors from a large PDF file by creating a chain of the `UTL_TO_TEXT`, `UTL_TO_CHUNKS`, and `UTL_TO_EMBEDDINGS` chainable utility functions. 

As shown in the following diagram, a *file-to-text-to-chunks-to-embeddings* chain performs a set of operations in this order: 

  1. Converts a PDF file to a plain text file by calling `UTL_TO_TEXT`. 
  2. Splits the resulting text into many appropriate-sized chunks by calling `UTL_TO_CHUNKS`. 
  3. Generates vector embeddings on each chunk by calling `UTL_TO_EMBEDDINGS`. 



![Description of pdf_to_text_to_chunks_to_embeddings.eps follows](img/pdf_to_text_to_chunks_to_embeddings.jpg)  


Common Use Cases

Let us look at some common use cases to understand how you can customize and apply these transformation chains:

**Single-Step or Direct Transformation**: 

  * **Document to vectors**: 

As discussed earlier, a common use case is to automatically generate vector embeddings from a set of documents. Embeddings are created by vector embedding models, and are numerical representations that capture the semantic meaning of your input data (as explained in [Overview of Oracle AI Vector Search](overview-ai-vector-search.md#GUID-746EAA47-9ADA-4A77-82BB-64E8EF5309BE)). The resulting embeddings are high-dimensional vectors, which you can use for text classification, question-answering, information retrieval, or text processing applications. 

    * You can convert a set of documents to plain text, split the resulting text into smaller chunks to finally generate embeddings on each chunk, in a single *file-to-text-to-chunks-to-embeddings* chain. See [Perform Chunking With Embedding](perform-chunking-embedding.md#GUID-9CB40305-392A-4CB0-AD49-7730504BEC18). 

    * For smaller documents or text strings, you can eliminate chunking (as explained in [Understand the Stages of Data Transformations](understand-stages-data-transformations.md#GUID-D6F1E7B6-5642-46A2-B84D-B8E37E4C353E)) and directly generate embeddings in a single *text-embeddings* chain. See [Generate Embeddings](generate-embeddings.md#GUID-813E0E54-9EEF-43FA-A506-1F276D47E7A6). 

  * **Document to vectors, with chunking and summarization**: 

Another use case might be to generate a short summary of a document and then automatically extract vectors from that summary.

After generating the summary, you can either generate a single vector (using `UTL_TO_EMBEDDING`) or chunk it and then generate multiple vectors (using `UTL_TO_EMBEDDINGS`). 

    * You can convert the document to plain text, summarize the text into a concise gist to finally create a single embedding on the summarized text, in a *file-to-text-to-summary-to-embedding* chain. 

    * You can convert the document to plain text, summarize the text into a gist, split the gist into chunks to finally create multiple embeddings on each summarized chunk, in a *file-to-text-to-summary-to-chunks-to-embeddings* chain. 

While both the chunking and summarization techniques make text smaller, they do so in different ways. Chunking just breaks the text into smaller pieces, whereas summarization extracts a salient meaning and context of that text into free-form paragraphs or bullet points. 

By summarizing the entire document and appending the summary to each chunk, you get the best of both worlds, that is, an individual piece that also has a high-level understanding of the overall document.

  * **Image to vectors**: 

You can directly generate a vector embedding based on an image, which can be used for image classification, object detection, comparison of large datasets of images, or for a more effective similarity search on documents that include images. Embeddings are created by image embedding models or multimodal embedding models that extract each visual element of an image (such as shape, color, pattern, texture, action, or object) and accordingly assign a numerical value to each element.

See [Convert Image to Embedding Using Public REST Providers](convert-image-embedding-using-public-third-party-apis.md#GUID-1F06AAA4-7333-40F5-AF43-FE124A7378A4). 




**Step-by-Step or Parallel Transformation**: 

  * **Text to vector**: 

A common use case might be information retrieval applications or chatbots, where you can convert a user's natural language text query string to a query vector on the fly. You can then compare that query vector against the existing vectors for a fast similarity search.

You can vectorize a query vector and then run that query vector against your index vectors, in a *text-to-embedding* chain. 

See [Convert Text String to Embedding Within Oracle Database](convert-text-string-embedding-oracle-database.md#GUID-1F9E9C73-6118-427E-8FB7-D4EBCCECC6D0), [Convert Text String to Embedding Using Public REST Providers](convert-text-string-embedding-using-public-third-party-apis.md#GUID-C693ED29-A963-4F19-BF2B-028545A3AA00), and [Convert Text String to Embedding Using the Local REST Provider Ollama](convert-text-string-embedding-locally-ollama.md#GUID-A77CD2EC-7DF5-4E4C-8AD8-4C274E5BA8E8). 

  * **Text to chunks**: 

Another use case might be to build a standalone Text Chunking system to break down a large amount of text into smaller yet semantically meaningful pieces, in a *text-to-chunks* chain. 

This method also gives you more flexibility to experiment with your chunks, where you can create, inspect, and accordingly amend the chunking results and then proceed further.

See [Convert Text to Chunks With Custom Chunking Specifications](convert-text-chunks-custom-chunking-specifications.md#GUID-E6771B78-5FB4-4EBA-92F5-BE9FF2DF6AFA) and [Convert File to Text to Chunks to Embeddings Within Oracle Database](convert-file-text-chunks-embeddings-oracle-database.md#GUID-10D0A76C-2DAE-42BE-A332-3EEF6D91D701). 

  * **Prompt or media file to text**: 

You can communicate with Large Language Models (LLMs) to perform several language-related tasks, such as text generation, translation, summarization, or Q&A. You can input text data in natural language (for prompts) or media data (for images) to an LLM-powered chat interface. LLM then processes this information to generate a text response.

    * **Prompt to text**: 

A prompt can be an input text string, such as a question that you ask an LLM. For example, "`What is Oracle Text?`". A prompt can also be a set of instructions or a command, such as "`Summarize the following ...`", "`Draft an email asking for ...`", or "`Rewrite the following ...`". The LLM responds with a textual answer, description, or summary based on the specified task in the prompt. 

See [Generate Text Using Public REST Providers](generate-text-using-public-third-party-apis.md#GUID-74B1916B-8B3F-43A0-9C80-53A068A639D2) and [Generate Text Using the Local REST Provider Ollama](generate-text-locally-ollama.md#GUID-35077FAC-0304-42D9-BA4B-F56EBEC7E35F). 

    * **Image to text**: 

You can also prompt with a media file (such as an image) to generate a textual analysis or description of the contents of the image, which can then be used for image classification, object detection, or similarity search. 

Here, you additionally supply a text question as the prompt along with the image. For example, the prompt can be "`What is this image about?`" or "`How many birds are there in this image?`". 

See [Describe Images Using Public REST Providers](describe-images-using-public-third-party-apis.md#GUID-D386049F-45B4-4047-B1E9-0A5955F9F1BA). 

    * **Text to summary**: 

You can build a standalone Text Summarization system to generate a summary of large or complex documents, in a *text-to-summary* chain. 

This method also gives you more flexibility to experiment with your summaries, where you can create, inspect, and accordingly amend the summarization results and then proceed further.

See [Generate Summary Using Public REST Providers](generate-summary-using-public-third-party-apis.md#GUID-915112EE-3681-4765-8B3F-F21313EE4878). 

> **note:** In addition to using external LLMs for summarization, you can use Oracle Text to generate a gist (summary) within the database. See [UTL_TO_SUMMARY](utl_to_summary.md#GUID-EC9DDB58-6A15-4B36-BA66-ECBA20D2CE57). 

    * **RAG implementation**

A prompt can include results from a search, through the implementation of Retrieval Augmented Generation (RAG). Using RAG can mitigate inaccuracies and hallucinations, which may occur in generated responses when using LLMs. 

See [SQL RAG Example](sql-rag-example.md#GUID-C859F916-A03E-4609-8C65-0B792ADB91FB). 

You can additionally utilize reranking to enhance the quality of information ingested into an LLM. See [Use Reranking for Better RAG Results](use-reranking-better-rag-results.md#GUID-969CE0B6-FF30-46F0-AADC-90B406F4F645). 




Schedule Vector Utility Packages

Some of the transformation chains may take a long time depending on your workload and implementation, thus you can schedule to run Vector Utility PL/SQL packages in the background.

The `DBMS_SCHEDULER` PL/SQL package helps you effectively schedule these packages, without manual intervention. 

For information on how to create, run, and manage jobs with Oracle Scheduler, see [*Oracle Database Administrator’s Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ADMIN-GUID-CAC644C1-854D-44FB-8FF6-BE6DDE809B4F). 

**Parent topic:** [About PL/SQL Packages to Generate Embeddings](pl-sql-packages-generate-embeddings.md)
