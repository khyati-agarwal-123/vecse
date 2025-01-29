## DBMS_VECTOR {#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12}

The `DBMS_VECTOR` package simplifies common operations with Oracle AI Vector Search, such as extracting chunks or embeddings from user data, generating text for a given prompt or an image, creating a vector index, or reporting on index accuracy. 

This table lists the `DBMS_VECTOR` subprograms and briefly describes them. 

**Table: DBMS_VECTOR Package Subprograms**

Subprogram | Description  
---|---  
**ONNX Model Related Procedures**: <br>These procedures enable you to load an ONNX model into Oracle Database and drop the ONNX model.  
[LOAD_ONNX_MODEL](load_onnx_model-procedure.md#GUID-7F1D7992-D8F7-4AD9-9BF6-6EFFC1B0617A) |  Loads an ONNX model into the database  
[LOAD_ONNX_MODEL_CLOUD](load_onnx_model_cloud.md#GUID-82A8D291-8096-4A7C-8882-9B6AC4A7FCCB) | Loads an ONNX model from object storage into the database  
[DROP_ONNX_MODEL Procedure](drop_onnx_model-procedure-dbms_vector.md#GUID-8031CBC3-3B94-4780-9647-C7D55C0B3331) |  Drops the ONNX model  
**Chainable Utility (UTL) Functions**: <br>These functions are a set of modular and flexible functions within vector utility PL/SQL packages. You can chain these together to automate end-to-end data transformation and similarity search operations.  
[UTL_TO_CHUNKS](utl_to_chunks-dbms_vector.md#GUID-4D96238E-2ABD-450D-8A9A-65E16A33481A) |  Splits data into smaller pieces or chunks  
[UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector.md#GUID-8E615832-F6C0-4435-8F43-3FAF80692D5B) |  Converts text or an image to one or more vector embeddings  
[UTL_TO_GENERATE_TEXT](utl_to_generate_text-dbms_vector.md#GUID-EA78DFB6-D951-43D1-8ECB-DD6D21C6F6A6) |  Generates text for a prompt (input string) or an image  
**Credential Helper Procedures**: <br>These procedures enable you to securely manage authentication credentials in the database. You require these credentials to enable access to third-party service providers for making REST calls.  
[CREATE_CREDENTIAL](create_credential-dbms_vector.md#GUID-4BBCBF46-3903-4EBB-8BE8-A7690151CF25) |  Creates a credential name  
[DROP_CREDENTIAL](drop_credential-dbms_vector.md#GUID-9F5DA186-6F2E-471F-9CEC-B32D502A1695) |  Drops an existing credential name  
**Data Access Functions**: <br>These functions enable you to retrieve data, create index, and perform simple similarity search operations.   
[CREATE_INDEX](create_index.md#GUID-33B844C5-CF57-4213-B2C8-31D82D2E02F6) |  Creates a vector index  
[REBUILD_INDEX](rebuild_index.md#GUID-7A7B63F6-97E3-44C6-8CD4-C84376F06F14) |  Rebuilds a vector index  
[GET_INDEX_STATUS](get_index_status.md#GUID-37E977D9-E8A8-4BDD-B6ED-69D68DCAF5D0) |  Describes the status of a vector index creation  
[ENABLE_CHECKPOINT](enable_checkpoint.md#GUID-B6B44353-4AE5-4A3F-BD5F-2F014A642014) |  Enables the Checkpoint feature for a vector index user and index name  
[DISABLE_CHECKPOINT](disable_checkpoint.md#GUID-BD7B95F2-5D35-4D94-9A34-ECE37D067C73) |  Disables the Checkpoint feature for a vector index user and index name  
[INDEX_VECTOR_MEMORY_ADVISOR](index_vector_memory_advisor.md#GUID-68009857-9F68-4894-8341-46EF81975A1B) |  Determines the vector memory size that is needed for a vector index  
[QUERY](query.md#GUID-BCB1DEE6-E8E0-43EA-9E5F-E346C981BC05) |  Performs a similarity search query  
[RERANK](rerank-dbms_vector.md#GUID-E821AF93-C180-4CEB-BE4F-2339B07ED2FE) |  Reorders search results for a more relevant output  
**Accuracy Reporting Function**: <br>These functions enable you to determine the accuracy of existing search indexes and to capture accuracy values achieved by approximate searches performed by past workloads.  
[INDEX_ACCURACY_QUERY](index_accuracy_query.md#GUID-4DC6EDAD-659E-4093-B1CE-D116E0EC6A1D) |  Verifies the accuracy of a vector index  
[INDEX_ACCURACY_REPORT](index_accuracy_report.md#GUID-E4360384-B882-4D5C-BC7F-332BB5E54516) |  Captures accuracy values achieved by approximate searches  
  
> **note:** 

`DBMS_VECTOR` is a lightweight package that does not support text processing or summarization operations. Therefore, the `UTL_TO_TEXT` and `UTL_TO_SUMMARY` chainable utility functions and all the chunker helper procedures are available only in the advanced `DBMS_VECTOR_CHAIN` package. 

  * [CREATE_CREDENTIAL](create_credential-dbms_vector.md)  
Use the `DBMS_VECTOR.CREATE_CREDENTIAL` credential helper procedure to create a credential name for storing user authentication details in Oracle Database. 
  * [CREATE_INDEX](create_index.md)  
Use the `DBMS_VECTOR.CREATE_INDEX` procedure to create a vector index. 
  * [DISABLE_CHECKPOINT](disable_checkpoint.md)  
Use the `DISABLE_CHECKPOINT` procedure to disable the Checkpoint feature for a given Hierarchical Navigable Small World (HNSW) index user and HNSW index name. This operation purges all older checkpoints for the HNSW index. It also disables the creation of future checkpoints as part of the HNSW graph refresh. 
  * [DROP_CREDENTIAL](drop_credential-dbms_vector.md)  
Use the `DBMS_VECTOR.DROP_CREDENTIAL` credential helper procedure to drop an existing credential name from the data dictionary. 
  * [DROP_ONNX_MODEL Procedure](drop_onnx_model-procedure-dbms_vector.md)  
This procedure deletes the specified ONNX model. 
  * [ENABLE_CHECKPOINT](enable_checkpoint.md)  
Use the `ENABLE_CHECKPOINT` procedure to enable the Checkpoint feature for a given Hierarchical Navigable Small World (HNSW) index user and HNSW index name. 
  * [GET_INDEX_STATUS](get_index_status.md)  
Use the `GET_INDEX_STATUS` procedure to query the status of a vector index creation. 
  * [INDEX_ACCURACY_QUERY](index_accuracy_query.md)  
Use the `DBMS_VECTOR.INDEX_ACCURACY_QUERY` function to verify the accuracy of a vector index for a given query vector, top-K, and target accuracy. 
  * [INDEX_ACCURACY_REPORT](index_accuracy_report.md)  
Use the `DBMS_VECTOR.INDEX_ACCURACY_REPORT` function to capture from your past workloads, accuracy values achieved by approximate searches using a particular vector index for a certain period of time. 
  * [INDEX_VECTOR_MEMORY_ADVISOR](index_vector_memory_advisor.md)  
Use the `INDEX_VECTOR_MEMORY_ADVISOR` procedure to determine the vector memory size needed for a particular vector index. This helps you evaluate the number of indexes that can fit for each simulated vector memory size. 
  * [LOAD_ONNX_MODEL](load_onnx_model-procedure.md)  
This procedure enables you to load an ONNX model into the Database. 
  * [LOAD_ONNX_MODEL_CLOUD](load_onnx_model_cloud.md)  
This procedure enables you to load an ONNX model from object storage into the Database. 
  * [QUERY](query.md)  
Use the `DBMS_VECTOR.QUERY` function to perform a similarity search operation which returns the top-k results as a JSON array. 
  * [REBUILD_INDEX](rebuild_index.md)  
Use the `DBMS_VECTOR.REBUILD_INDEX` function to rebuild a vector index. 
  * [RERANK](rerank-dbms_vector.md)  
Use the `DBMS_VECTOR.RERANK` function to reassess and reorder an initial set of results to retrieve more relevant search output. 
  * [UTL_TO_CHUNKS](utl_to_chunks-dbms_vector.md)  
Use the `DBMS_VECTOR.UTL_TO_CHUNKS` chainable utility function to split a large plain text document into smaller chunks of text. 
  * [UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector.md)  
Use the `DBMS_VECTOR.UTL_TO_EMBEDDING` and `DBMS_VECTOR.UTL_TO_EMBEDDINGS` chainable utility functions to generate one or more vector embeddings from textual documents and images. 
  * [UTL_TO_GENERATE_TEXT](utl_to_generate_text-dbms_vector.md)  
Use the `DBMS_VECTOR.UTL_TO_GENERATE_TEXT` chainable utility function to generate a text response for a given prompt or an image, by accessing third-party text generation models. 



**Parent topic:** [Vector Search PL/SQL Packages](vector-search-pl-sql-packages-node.md)
