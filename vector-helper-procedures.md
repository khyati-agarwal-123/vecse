## About Vector Helper Procedures {#GUID-E97EE03F-C922-4C2D-82A7-38152CBF70EB}

Vector helper procedures let you configure authentication credentials, preferences, and language-specific data for use in chainable utility functions.

At a high level, the supplied vector helper procedures include:

  * **Credential helper procedures** to securely manage authentication credentials, which are used to access third-party providers when making REST API calls. 

Function | Description  
---|---  
`CREATE_CREDENTIAL` |  Creates a credential name for securely storing user authentication credentials in Oracle Database.   
`DROP_CREDENTIAL` |  Drops an existing credential name.  
  
  * **Preference helper procedures** to manage vectorizer preferences, which are used when creating or altering hybrid vector indexes. 

Function | Description  
---|---  
`CREATE_PREFERENCE` |  Creates a vectorizer preference in the database.  
`DROP_PREFERENCE` |  Drops an existing vectorizer preference from the database.  
  
  * **Chunker helper procedures** to manage custom vocabulary and language data, which are used when chunking user data. 

Function | Description  
---|---  
`CREATE_VOCABULARY` |  Loads your own vocabulary file into the database.   
`DROP_VOCABULARY` |  Removes the specified vocabulary data from the database.  
`CREATE_LANG_DATA` |  Loads your own language data file (abbreviation tokens) into the database.   
`DROP_LANG_DATA` |  Removes abbreviation data for a given language from the database.  
  



**Related Topics**

  * [Vector Search PL/SQL Packages](vector-search-pl-sql-packages-node.md#GUID-04ACF179-957C-4F03-AC1D-4DA44B3E12A2)
  * [Text Processing Views](text-processing-views.md#GUID-E2B9F02C-E2A6-439B-9A2E-177FF7FA6EE0)



**Parent topic:** [About PL/SQL Packages to Generate Embeddings](pl-sql-packages-generate-embeddings.md)
