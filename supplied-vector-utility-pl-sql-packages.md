## Supplied Vector Utility PL/SQL Packages {#GUID-D73320C0-C2E3-4B67-A7C9-C0490DC9DE6C}

Use either a lightweight `DBMS_VECTOR` package or a more advanced `DBMS_VECTOR_CHAIN` package with full capabilities. 

  * `DBMS_VECTOR`: 

This package simplifies common operations with Oracle AI Vector Search, such as chunking text into smaller segments, extracting vector embeddings from user data, or generating text for a given prompt. 

Subprogram | Operation | Provider | Implementation  
---|---|---|---  
Chainable Utility Functions |  `UTL_TO_CHUNKS` to perform chunking  |  Oracle Database  |  Calls the `VECTOR_CHUNKS` SQL function under the hood   
`UTL_TO_EMBEDDING` and `UTL_TO_EMBEDDINGS` to generate one or more embeddings  |  Oracle Database |  Calls the embedding model in ONNX format stored in the database  
Third-party REST providers |  Calls the specified third-party embedding model  
`UTL_TO_GENERATE_TEXT` to generate text for prompts and images  |  Third-party REST providers |  Calls the specified third-party text generation model  
`RERANK` to retrieve more relevant search output  |  Third-party REST providers |  Calls the specified third-party embedding model  
Credential Helper Procedures |  `CREATE_CREDENTIAL` and `DROP_CREDENTIAL` to manage credentials for third-party service providers  |  Oracle Database  |  Stores credentials securely for use in Chainable Utility Functions  
  
For detailed information on this package, see [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12). 

  * `DBMS_VECTOR_CHAIN`: 

This package provides chunking and embedding functions along with some text generation and summarization capabilities. It is more suitable for text processing with similarity search and hybrid search, using functionality that can be pipelined together for an end-to-end search.

Subprogram | Operation | Provider | Implementation  
---|---|---|---  
Chainable Utility Functions |  `UTL_TO_TEXT` to extract plain text data from documents  |  Oracle Database  |  Uses the Oracle Text component (`CONTEXT`) of Oracle Database   
`UTL_TO_CHUNKS` to perform chunking  |  Oracle Database  |  Calls the `VECTOR_CHUNKS` SQL function under the hood   
`UTL_TO_EMBEDDING` and `UTL_TO_EMBEDDINGS` to generate one or more embeddings  |  Oracle Database  |  Calls the embedding model in ONNX format stored in the database  
Third-party REST providers |  Calls the specified third-party embedding model  
`UTL_TO_SUMMARY` to generate summaries  |  Oracle Database |  Uses Oracle Text  
Third-party REST providers |  Calls the specified third-party text summarization model  
`UTL_TO_GENERATE_TEXT` to generate text for prompts and images  |  Third-party REST providers |  Calls the specified third-party text generation model  
`RERANK` to retrieve more relevant search output  |  Third-party REST providers |  Calls the specified third-party embedding model  
Credential Helper Procedures |  `CREATE_CREDENTIAL` and `DROP_CREDENTIAL` to manage credentials for third-party service providers  |  Oracle Database  |  Stores credentials securely for use in Chainable Utility Functions  
Preference Helper Procedures | `CREATE_PREFERENCE` and `DROP_PREFERENCE` to manage preferences for hybrid vector indexes  |  Oracle Database  |  Creates vectorizer preferences for use in hybrid vector indexing pipelines  
Chunker Helper Procedures |  `CREATE_VOCABULARY` and `DROP_VOCABULARY` to manage custom token vocabularies  |  Oracle Database  |  Uses Oracle Text  
`CREATE_LANG_DATA` and `DROP_LANG_DATA` to manage language-specific data (abbreviation tokens)  |  Oracle Database  |  Uses Oracle Text  
  
> **note:** 

The `DBMS_VECTOR_CHAIN` package requires you to install the `CONTEXT` component of Oracle Text, an Oracle Database technology that provides indexing, term extraction, text analysis, text summarization, word and theme searching, and other utilities. 

Due to underlying dependance on the text processing capabilities of Oracle Text, note that both the `UTL_TO_TEXT` and `UTL_TO_SUMMARY` chainable utility functions and all the chunker helper procedures are available only in this package through Oracle Text. 

For detailed information on this package, see [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504). 




**Related Topics**

  * [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09)



**Parent topic:** [About PL/SQL Packages to Generate Embeddings](pl-sql-packages-generate-embeddings.md)
