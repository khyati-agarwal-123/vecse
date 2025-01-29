## Convert File to Embeddings Within Oracle Database {#GUID-04C88292-0A04-44F6-8FB4-DDCA5CE38856}

Directly extract vector embeddings from a PDF document, using a single-step statement, by accessing a vector embedding model stored in the database.

Here, you perform a file-to-text-to-chunks-to-embeddings transformation by calling a set of `DBMS_VECTOR_CHAIN.UTL` functions in a single `CREATE TABLE` statement. 

This statement creates a relational table (`doc_chunks`) from unstructured text chunks and the corresponding vector embeddings: 
```
    CREATE TABLE doc_chunks as
    (select dt.id doc_id, et.embed_id, et.embed_data, to_vector(et.embed_vector) embed_vector
    from
    documentation_tab dt,
    dbms_vector_chain.utl_to_embeddings(
    dbms_vector_chain.utl_to_chunks(dbms_vector_chain.utl_to_text(dt.data), json('{"normalize":"all"}')),
    json('{"provider":"database", "model":"doc_model"}')) t,
    JSON_TABLE(t.column_value, '$[*]' COLUMNS (embed_id NUMBER PATH '$.embed_id', embed_data VARCHAR2(4000) PATH '$.embed_data', embed_vector CLOB PATH '$.embed_vector')) et
    );
```
    

Note that each successive function depends on the output of the previous function, so the order of chains is important here. First, the output from `utl_to_text` (`dt.data` column) is passed as an input for `utl_to_chunks` and then the output from `utl_to_chunks` is passed as an input for `utl_to_embeddings`. 

For complete example, run [SQL Quick Start Using a Vector Embedding Model Uploaded into the Database](sql-quick-start-using-vector-embedding-model-uploaded-database.md#GUID-403EB84E-3047-4905-844C-BD4A8670B8A4), where you can see how to embed Oracle Database Documentation books in the `doc_chunks` table and perform similarity searches using vector indexes. 

**Parent topic:** [Perform Chunking With Embedding](perform-chunking-embedding.md)
