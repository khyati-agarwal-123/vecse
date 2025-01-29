## About SQL Functions to Generate Embeddings {#GUID-AEAA4AA9-2D47-4451-B6D5-E505C133D9CF}

Choose to implement Vector Utility SQL functions to perform parallel or on-the-fly chunking and embedding operations, within the database. The supplied SQL functions for vector generation are `VECTOR_CHUNKS` and `VECTOR_EMBEDDING`. 

Vector Utility SQL functions are intended for a direct and quick interaction with data, within pure SQL.

VECTOR_CHUNKS

Use the `VECTOR_CHUNKS` SQL function if you want to split plain text into chunks (pieces of words, sentences, or paragraphs) in preparation for the generation of embeddings, to be used with a vector index. 

For example, you can use this function to build a standalone Text Chunking system that lets you break down a large PDF document into smaller yet semantically meaningful chunk texts. You can experiment with your chunks by running parallel chunking operations, where you can inspect each chunk text, accordingly amend the chunking results, and then proceed further with other data transformation stages.

To generate chunks, this function uses the in-house implementation with Oracle Database. 

For detailed information on this function, see [VECTOR_CHUNKS](vector_chunks.md#GUID-5927E2FA-6419-4744-A7CB-3E62DBB027AD). 

VECTOR_EMBEDDING

Use the `VECTOR_EMBEDDING` function if you want to generate a single vector embedding for different data types. 

For example, you can use this function in information-retrieval applications or chatbots, where you want to generate a query vector on the fly from a user's natural language text input string. You can then query a vector field with this query vector for a fast similarity search.

To generate an embedding, this function uses a vector embedding model (in ONNX format) that you load into the database.

> **note:** If you want to generate embeddings by using third-party vector embedding models, then use Vector Utility PL/SQL packages. These packages let you work with both embedding models (in ONNX format) stored in the database and third-party embedding models (by calling third-party REST APIs). 

For detailed information on this function, see [VECTOR_EMBEDDING](vector_embedding.md#GUID-5ED78260-6D21-4B6B-86E0-A1E70EFA11CA). 

**Related Topics**

  * [Import Pretrained Models in ONNX Format](import-pretrained-models-onnx-format-vector-generation-database.md#GUID-D8140BF9-08E9-4B3F-9E28-E40A6FD181A4)
  * [Generate Embeddings](generate-embeddings.md#GUID-813E0E54-9EEF-43FA-A506-1F276D47E7A6)



**Parent topic:** [About Vector Generation](vector-generation.md)
