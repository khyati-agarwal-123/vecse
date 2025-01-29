## Convert Text to Chunks With Custom Chunking Specifications {#GUID-E6771B78-5FB4-4EBA-92F5-BE9FF2DF6AFA}

A chunked output, especially for long and complex documents, sometimes loses contextual meaning or coherence with its parent content. In this example, you can see how to refine your chunks by applying custom chunking specifications. 

Here, you use the `VECTOR_CHUNKS` SQL function or the `UTL_TO_CHUNKS()` PL/SQL function from the `DBMS_VECTOR_CHAIN` package. 

> **note:** This example shows how to apply some of the custom chunking parameters. To further explore all the supported chunking parameters with more detailed examples, see [Explore Chunking Techniques and Examples](explore-chunking-techniques-and-examples.md#GUID-BFDE8B0A-6302-472C-AD8B-1BEB9AA2CB87). 

  1. Connect to Oracle Database as a local user.
    1. Log in to SQL*Plus as the `SYS` user, connecting as `SYSDBA`:
```
        conn sys/password as sysdba
```
```
        CREATE TABLESPACE tbs1
        DATAFILE 'tbs5.dbf' SIZE 20G AUTOEXTEND ON
        EXTENT MANAGEMENT LOCAL
        SEGMENT SPACE MANAGEMENT AUTO;
```
```
        SET ECHO ON
        SET FEEDBACK 1
        SET NUMWIDTH 10
        SET LINESIZE 80
        SET TRIMSPOOL ON
        SET TAB OFF
        SET PAGESIZE 10000
        SET LONG 10000
```
        

    2. Create a local test user (`docuser`) and grant necessary privileges:
```
        DROP USER docuser cascade;
```
```
        CREATE USER docuser identified by docuser DEFAULT TABLESPACE tbs1 quota unlimited on tbs1;
```
```
        GRANT DB_DEVELOPER_ROLE to docuser;
```
        

    3. Connect as the local user (`docuser`):
```
        CONN docuser/password
```
        

  2. Create a relational table (`documentation_tab`) to store unstructured text chunks in it:
```
    DROP TABLE IF EXISTS documentation_tab;
    
    CREATE TABLE documentation_tab (
    id NUMBER,
    text VARCHAR2(2000));
```
```
    INSERT INTO documentation_tab VALUES(1,
    'Oracle AI Vector Search stores and indexes vector embeddings'||
    ' for fast retrieval and similarity search.'||CHR(10)||CHR(10)||
    '    About Oracle AI Vector Search'||CHR(10)||
    '    Vector Indexes are a new classification of specialized indexes'||
    ' that are designed for Artificial Intelligence (AI) workloads that allow'||
    ' you to query data based on semantics, rather than keywords.'||CHR(10)||CHR(10)||
    '    Why Use Oracle AI Vector Search?'||CHR(10)||
    ' The biggest benefit of Oracle AI Vector Search is that semantic search'||
    ' on unstructured data can be combined with relational search on business'||
    ' data in one single system.'||CHR(10));
    
    COMMIT;
```
```
    SET LINESIZE 1000;
    SET PAGESIZE 200;
    COLUMN doc FORMAT 999;
    COLUMN id  FORMAT 999;
    COLUMN pos FORMAT 999;
    COLUMN siz FORMAT 999;
    COLUMN txt FORMAT a60;
    COLUMN data FORMAT a80;
```
    

  3. Call the `VECTOR_CHUNKS` SQL function and specify the following custom chunking parameters. Setting the `NORMALIZE` parameter to `all` ensures that the formatting of the results is easier to read, which is especially helpful when the data includes PDF documents:
```
    SELECT D.id doc, C.chunk_offset pos, C.chunk_length siz, C.chunk_text txt
    FROM documentation_tab D, VECTOR_CHUNKS(D.text
    BY words
    MAX 50
    OVERLAP 0
    SPLIT BY recursively
    LANGUAGE american
    NORMALIZE all) C;
```
    

To call the same operation using the `UTL_TO_CHUNKS` function from the `DBMS_VECTOR_CHAIN` package, run: 
```
    SELECT D.id doc,
    JSON_VALUE(C.column_value, '$.chunk_id' RETURNING NUMBER) AS id,
    JSON_VALUE(C.column_value, '$.chunk_offset' RETURNING NUMBER) AS pos,
    JSON_VALUE(C.column_value, '$.chunk_length' RETURNING NUMBER) AS siz,
    JSON_VALUE(C.column_value, '$.chunk_data') AS txt
    FROM documentation_tab D,
    dbms_vector_chain.utl_to_chunks(D.text,
    JSON('{"by":"words",
    "max":"50",
    "overlap":"0",
    "split":"recursively",
    "language":"american",
    "normalize":"all"}')) C;
```
    

This returns a set of three chunks, which are split by words recursively using blank lines, new lines, and spaces:
```
    DOC  POS  SIZ  TXT
    ---- ---- ---- ------------------------------------------------------------
    1   1    108   Oracle AI Vector Search stores and indexes vector embeddings
    for fast retrieval and similarity search.
    
    1   109  234   About Oracle AI Vector Search
    Vector Indexes are a new classification of specialized index
    es that are designed for Artificial Intelligence (AI) worklo
    ads that allow you to query data based on semantics, rather
    than keywords.
    
    1   343  204   Why Use Oracle AI Vector Search?
    The biggest benefit of Oracle AI Vector Search is that seman
    tic search on unstructured data can be combined with relatio
    nal search on business data in one single system.
```
    

The chunking results contain: 
     * `chunk_id` as `DOC`: ID for each chunk 

     * `chunk_offset` as `POS`: Original position of each chunk in the source document, relative to the start of document (which has a position of `1`) 

     * `chunk_length` as `SIZ`: Character length of each chunk 

     * `chunk_data` as `TXT`: Textual content from each chunk 




**Related Topics**

  * [VECTOR_CHUNKS](vector_chunks.md#GUID-5927E2FA-6419-4744-A7CB-3E62DBB027AD)
  * [UTL_TO_CHUNKS](utl_to_chunks-dbms_vector_chain.md#GUID-4E145629-7098-4C7C-804F-FC85D1F24240)



**Parent topic:** [Perform Chunking With Embedding](perform-chunking-embedding.md)
