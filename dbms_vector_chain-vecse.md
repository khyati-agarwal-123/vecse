## DBMS_VECTOR_CHAIN {#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504}

The `DBMS_VECTOR_CHAIN` package enables advanced operations with Oracle AI Vector Search, such as chunking and embedding data along with text generation and summarization capabilities. It is more suitable for text processing with similarity search and hybrid search, using functionality that can be pipelined together for an end-to-end search. 

This table lists the `DBMS_VECTOR_CHAIN` subprograms and briefly describes them. 

**Table: DBMS_VECTOR_CHAIN Package Subprograms**

Subprogram | Description  
---|---  
**Chainable Utility (UTL) Functions**: <br>These functions are a set of modular and flexible functions within vector utility PL/SQL packages. You can chain these together to automate end-to-end data transformation and similarity search operations.  
[UTL_TO_TEXT](utl_to_text.md#GUID-73397E89-92FB-48ED-94BB-1AD960C4EA1F) |  Extracts plain text data from documents  
[UTL_TO_CHUNKS](utl_to_chunks-dbms_vector_chain.md#GUID-4E145629-7098-4C7C-804F-FC85D1F24240) |  Splits data into smaller pieces or chunks  
[UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector_chain.md#GUID-C6439E94-4E86-4ECD-954E-4B73D53579DE) |  Converts text or an image to one or more vector embeddings  
[UTL_TO_SUMMARY](utl_to_summary.md#GUID-EC9DDB58-6A15-4B36-BA66-ECBA20D2CE57) |  Extracts a summary from documents  
[UTL_TO_GENERATE_TEXT](utl_to_generate_text-dbms_vector_chain.md#GUID-017C9002-194C-48E5-B59B-EF5C60BC8405) |  Generates text for a prompt (input string) or an image  
**Credential Helper Procedures**: <br>These procedures enable you to securely manage authentication credentials in the database. You require these credentials to enable access to third-party service providers for making REST calls.  
[CREATE_CREDENTIAL](create_credential-dbms_vector_chain.md#GUID-A6E28402-DC43-44C6-A1B2-75C3F270DD76) |  Creates a credential name  
[DROP_CREDENTIAL](drop_credential-dbms_vector_chain.md#GUID-FCA739BA-4849-45BF-A860-251D45E31B17) |  Drops an existing credential name  
**Preference Helper Procedures**: <br>These procedures enable you to manage vectorizer preferences, to be used with the `CREATE_HYBRID_VECTOR_INDEX` and `ALTER_INDEX` SQL statements when creating or managing hybrid vector indexes.   
[CREATE_PREFERENCE](create_preference.md#GUID-B83978CD-EAF8-4794-9652-F335C54C3385) |  Creates a vectorizer preference  
[DROP_PREFERENCE](drop_preference.md#GUID-72CCB0E1-2FB9-4F94-A3A5-6090E1927D3A) |  Drops an existing vectorizer preference  
**Chunker Helper Procedures**: <br>These procedures enable you to configure vocabulary and language data (abbreviations), to be used with the `VECTOR_CHUNKS` SQL function or `UTL_TO_CHUNKS` PL/SQL function.   
[CREATE_VOCABULARY](create_vocabulary.md#GUID-2D19528E-0F0D-4102-8EC7-E9EA62C66C2D) |  Loads your token vocabulary file into the database  
[DROP_VOCABULARY](drop_vocabulary.md#GUID-2CC9EBDB-717F-4AA7-8FDE-7503BE185D87) |  Removes existing vocabulary data  
[CREATE_LANG_DATA](create_lang_data.md#GUID-C9756FA9-B0B6-4750-8D9C-ADEF8B67C675) |  Loads your language data file into the database  
[DROP_LANG_DATA](drop_lang_data.md#GUID-55F54E07-1026-4021-A194-3E068471426E) |  Removes existing abbreviation data  
**Data Access Function**: <br>This function enables you to enhance search operations.   
[RERANK](rerank-dbms_vector_chain.md#GUID-08B0E5EE-B097-43CD-828C-05D45B70157D) |  Reorders search results for a more relevant output  
  
> **note:** 

The `DBMS_VECTOR_CHAIN` package requires you to install the `CONTEXT` component of Oracle Text, an Oracle Database technology that provides indexing, term extraction, text analysis, text summarization, word and theme searching, and other utilities. 

Due to underlying dependance on the text processing capabilities of Oracle Text, note that both the `UTL_TO_TEXT` and `UTL_TO_SUMMARY` chainable utility functions and all the chunker helper procedures are available only in this package through Oracle Text. 

  * [CREATE_CREDENTIAL](create_credential-dbms_vector_chain.md)  
Use the `DBMS_VECTOR_CHAIN.CREATE_CREDENTIAL` credential helper procedure to create a credential name for storing user authentication details in Oracle Database. 
  * [CREATE_LANG_DATA](create_lang_data.md)  
Use the `DBMS_VECTOR_CHAIN.CREATE_LANG_DATA` chunker helper procedure to load your own language data file into the database. 
  * [CREATE_PREFERENCE](create_preference.md)  
Use the `DBMS_VECTOR_CHAIN.CREATE_PREFERENCE` helper procedure to create a vectorizer preference, to be used when creating or updating hybrid vector indexes. 
  * [CREATE_VOCABULARY](create_vocabulary.md)  
Use the `DBMS_VECTOR_CHAIN.CREATE_VOCABULARY` chunker helper procedure to load your own token vocabulary file into the database. 
  * [DROP_CREDENTIAL](drop_credential-dbms_vector_chain.md)  
Use the `DBMS_VECTOR_CHAIN.DROP_CREDENTIAL` credential helper procedure to drop an existing credential name from the data dictionary. 
  * [DROP_LANG_DATA](drop_lang_data.md)  
Use the `DBMS_VECTOR_CHAIN.DROP_LANG_DATA` chunker helper procedure to remove abbreviation data from the data dictionary. 
  * [DROP_PREFERENCE](drop_preference.md)  
Use the `DBMS_VECTOR_CHAIN.DROP_PREFERENCE` preference helper procedure to remove an existing Vectorizer preference. 
  * [DROP_VOCABULARY](drop_vocabulary.md)  
Use the `DBMS_VECTOR_CHAIN.DROP_VOCABULARY` chunker helper procedure to remove vocabulary data from the data dictionary. 
  * [RERANK](rerank-dbms_vector_chain.md)  
Use the `DBMS_VECTOR_CHAIN.RERANK` function to reassess and reorder an initial set of results to retrieve more relevant search output. 
  * [UTL_TO_CHUNKS](utl_to_chunks-dbms_vector_chain.md)  
Use the `DBMS_VECTOR_CHAIN.UTL_TO_CHUNKS` chainable utility function to split a large plain text document into smaller chunks of text. 
  * [UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector_chain.md)  
Use the `DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDING` and `DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDINGS` chainable utility functions to generate one or more vector embeddings from textual documents and images. 
  * [UTL_TO_GENERATE_TEXT](utl_to_generate_text-dbms_vector_chain.md)  
Use the `DBMS_VECTOR_CHAIN.UTL_TO_GENERATE_TEXT` chainable utility function to generate a text response for a given prompt or an image, by accessing third-party text generation models. 
  * [UTL_TO_SUMMARY](utl_to_summary.md)  
Use the `DBMS_VECTOR_CHAIN.UTL_TO_SUMMARY` chainable utility function to generate a summary for textual documents. 
  * [UTL_TO_TEXT](utl_to_text.md)  
Use the `DBMS_VECTOR_CHAIN.UTL_TO_TEXT` chainable utility function to convert an input document (for example, PDF, DOC, JSON, XML, or HTML) to plain text. 
  * [Supported Languages and Data File Locations](supported-languages-and-data-file-locations.md)  
These are the supported languages for which language data files are distributed by default in the specified directories. 



**Parent topic:** [Vector Search PL/SQL Packages](vector-search-pl-sql-packages-node.md)
