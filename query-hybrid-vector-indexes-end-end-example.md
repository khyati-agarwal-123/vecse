## Query Hybrid Vector Indexes End-to-End Example {#GUID-4E340969-349E-4E46-AD98-AC2A0AEDB773}

In this example, you can see an end-to-end hybrid search workflow. First, you run the `CREATE HYBRID VECTOR INDEX` SQL statement that prepares, chunks, embeds, stores, and indexes your input data. You then perform vector search alongside keyword search using the `DBMS_HYBRID_VECTOR.SEARCH` PL/SQL query API. 

Here, you can see how to use hybrid search in an HR Recruitment scenario, where you want to hire employees with specific technical skills (keyword search for "C, Python, and Database") but who also display certain personality and leadership traits (semantic search for "prioritize teamwork and leadership experience"). You can also see how to perform different kinds of pure and non-pure hybrid search queries in multiple search modes.

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
        

    2. Create a local user (`docuser`) and grant necessary privileges:
```
        DROP USER docuser cascade;
```
```
        CREATE USER docuser identified by docuser DEFAULT TABLESPACE tbs1 quota unlimited on tbs1;
```
```
        GRANT DB_DEVELOPER_ROLE, create credential to docuser;
```
        

    3. Create a local directory on your server (`DEMO_DIR`) to store your input data and model files. Grant necessary privileges:
```
        create or replace directory DEMO_DIR as '/my_local_dir/';
```
```
        grant read, write on directory DEMO_DIR to docuser;
        
        commit;
```
        

    4. Connect as the local user (`docuser`):
```
        CONN docuser/password
```
        

  2. Load an ONNX format embedding model into Oracle Database by calling the `load_onnx_model` procedure.

This embedding model is internally used to generate vector embeddings from your input data.
```
    EXECUTE dbms_vector.drop_onnx_model(model_name => 'doc_model', force => true);
```
```
    EXECUTE dbms_vector.load_onnx_model(
    'DEMO_DIR', 'my_embedding_model.onnx', 'doc_model',
    JSON('{"function" : "embedding", "embeddingOutput" : "embedding" , "input": {"input": ["DATA"]}}'));
```
    

In this example, the procedure loads an ONNX model file, named `my_embedding_model.onnx` from the `DEMO_DIR` directory, into the database as `doc_model`. You must replace `my_embedding_model.onnx` with an ONNX export of your embedding model and `doc_model` with the name under which the imported model is stored in the database. 

> **note:** If you do not have an embedding model in ONNX format, then perform the steps listed in [ONNX Pipeline Models : Text Embedding](onnx-pipeline-models-text-embedding.md#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6). 

  3. Create a relational table (for example, `doc_tab`) and insert all your documents in a textual column (for example, `text`).
```
    DROP TABLE doc_tab purge;
```
```
    CREATE TABLE doc_tab (id number, text varchar2(500));
    
    insert into t1 values(1, 'Candidate-1: C Master. Optimizes low-level system (i.e. Database) performance with C. Strong leadership skills in guiding teams to deliver complex projects.');
    insert into t1 values(2, 'Candidate-2: Full-Stack Developer. Skilled in Database, C, HTML, JavaScript, and Python with experience in building responsive web applications. Thrives in collaborative team environments.');
    insert into t1 values(3, 'Candidate-3: DevOps Engineer. Manages CI/CD pipelines (Jenkins, Gitlab) with expertise in cloud infrastructure (OCI, AWS, GCP). Proven track record of streamlining deployments and ensuring high availability.');
    insert into t1 values(4, 'Candidate-4: Database Administrator (DBA). Maintains and secures enterprise database (Oracle, MySql, SQL Server). Passionate about data integrity and optimization. Strong mentor for junior DBA(s).');
    insert into t1 values(5, 'Candidate-5: C, Java, Python, and Database (DBA) Guru. Develops scalable applications. Strong leadership, teamwork and collaborative.');
    
    commit;
```
    

  4. Create a hybrid vector index (`my_hybrid_idx`) on the `text` column of the `doc_tab` table.
```
    CREATE HYBRID VECTOR INDEX my_hybrid_idx on
    doc_tab(text)
    parameters('model doc_model');
```
    

Here, `model` specifies the embedding model in ONNX format that you have imported into the database for generating embeddings. This is the only required indexing parameter to create a hybrid vector index. 

An index named `my_hybrid_idx` is created on the `text` column of the `doc_tab` table. The default vector index type is set to Inverted File Flat (IVF). 

  5. Query the hybrid vector index.

In the next steps, we will explore all possible ways in which you can query the hybrid vector index using multiple search modes described in [Understand Hybrid Search](understand-hybrid-search.md#GUID-310D2298-90F4-4AFE-AF03-F3B81E55F84C). You can then compare the results to examine the differences in scores and ranking for a comprehensive understanding. 

    1. **Simple default query**:

By default, hybrid search offers a simplified query with predefined fields. The minimum input parameters required are `hybrid_index_name` and `search_text`. 

The same search text (`C, Python, Database`) is used to query the vectorized chunk index and the text document index. This search text is transformed into a `CONTAINS` query for keyword search, and is vectorized for a `VECTOR_DISTANCE` query for semantic search. 
```
        select json_Serialize(
        dbms_hybrid_vector.search(
        json(
        '{
        "hybrid_index_name" : "my_hybrid_idx",
        "search_text"       : "C, Python, Database"
        }'
        )
        ) pretty)
        from dual;
```
        

This query returns the top 3 rows, ordered by score relevance. The highest final score (`69.54`) of Candidate-2 indicates the best match. All the return attributes are shown by default in this query. 
```
        [
        {
        "rowid"        : "AAAR9jAABAAAQeaAAB",
        "score"        : 69.54,
        "vector_score" : 69.69,
        "text_score"   : 68,
        "vector_rank"  : 1,
        "text_rank"    : 1,
        "chunk_text"   : "Candidate-2: Full-Stack Developer. Skilled in Database, C, HTM
        L, JavaScript, and Python with experience in building responsive web application
        s. Thrives in collaborative team environments.",
        "chunk_id"     : "1"
        },
        {
        "rowid"        : "AAAR9jAABAAAQeaAAA",
        "score"        : 62.77,
        "vector_score" : 65.55,
        "text_score"   : 35,
        "vector_rank"  : 4,
        "text_rank"    : 2,
        "chunk_text"   : "Candidate-1: C Master. Optimizes low-level system (i.e. Da
        tabase) performance with C. Strong leadership skills in guiding teams to deliver
        complex projects.",
        "chunk_id"     : "1"
        },
        {
        "rowid"        : "AAAR9jAABAAAQeaAAD",
        "score" : 62.15,
        "vector_score" : 68.17,
        "text_score"   : 2,
        "vector_rank"  : 2,
        "text_rank"    : 3,
        "chunk_text"   : "Candidate-4: Database Administrator (DBA). Maintains and
        secures enterprise database (Oracle, MySql, SQL Server). Passionate about data i
        ntegrity and optimization. Strong mentor for junior DBA(s).",
        "chunk_id"     : "1"
        }
        ]
```
        

    2. **Pure semantic in document mode (text as query input)**:

This is a pure vector-based similarity query to fetch document-level search results. Here, the `search_text` string is vectorized into a query vector for a `VECTOR_DISTANCE` semantic query. 

A vector search operates at the chunk level, where the query first extracts the top candidates of vectorized chunks, aggregates them by document using the `aggregator` (`MAX`) function, produces a combined score (according to the `aggregator`), and finally returns the top n documents. To know more about how this works, see [Pure Semantic in Document Mode](understand-hybrid-search.md#GUID-310D2298-90F4-4AFE-AF03-F3B81E55F84C__GUID-CA1E17B9-796C-4F54-B15B-71E9E9F5F9C2). 
```
        select json_Serialize(
        dbms_hybrid_vector.search(
        json(
        '{ "hybrid_index_name" : "my_hybrid_idx",
        "vector":
        {
        "search_text"   : "prioritize teamwork and leadership experience",
        "search_mode"   : "DOCUMENT",
        "aggregator"    : "MAX"
        },
        "return":
        {
        "values"        : [ "rowid", "score" ],
        "topN": 3
        }
        }'
        )
        ) pretty)
        from dual;
```
        

The top 3 documents are returned with the corresponding `ROWID`s of the `doc_tab` table rows as well as their vector scores. 
```
        [
        {
        "rowid" : "AAAR9jAABAAAQeaAAA",
        "score" : 61
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAD",
        "score" : 56.64
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAB",
        "score" : 55.75
        }
        ]
```
        

    3. **Pure semantic in document mode (embedding as query input)**:

As compared to the previous pure semantic query in document mode (where you used the `search_text` parameter to specify the query text for vector search), here you use the `search_vector` parameter to directly pass a vector embedding (query vector) as input for your vector search. 

In the same manner, this query retrieves the top vectorized chunk results, aggregates them by document using the `aggregator` (`MAX`) function, and finally returns the top n documents. 
```
        select json_Serialize(
        dbms_hybrid_vector.search(
        json(
        '{ "hybrid_index_name" : "my_hybrid_idx",
        "vector":
        {
        "search_vector"   : vector_serialize(
        vector_embedding(doc_model
        using
        "C, Python, Database"
        as data)
        RETURNING CLOB),
        "search_mode"     : "DOCUMENT",
        "aggregator"      : "MAX"
        },
        "return":
        {
        "values"     : [ "rowid", "score" ],
        "topN": 3
        }
        }'
        )
        ) pretty)
        from dual;
```
        

The top 3 documents are returned with the corresponding `ROWID`s of the `doc_tab` table rows as well as their vector scores. 
```
        [
        {
        "rowid" : "AAAR9jAABAAAQeaAAB",
        "score" : 69.69
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAD",
        "score" : 68.17
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAC",
        "score" : 67.48
        }
        ]
```
        

    4. **Pure semantic in chunk mode**:

This is a pure vector-based similarity query to retrieve chunk results. Note that this is the Oracle AI Vector Search semantic search equivalent. Here, vector search retrieves the best vectorized chunks from the same or different set of documents and then internally computes their vector scores. The top n chunks with the highest vector scores are returned. To know more about how this works, see [Pure Semantic in Chunk Mode](understand-hybrid-search.md#GUID-310D2298-90F4-4AFE-AF03-F3B81E55F84C__GUID-0611B161-9E85-449A-8574-0ECD1D156C60). 

As compared to the previous pure semantic search in document mode, aggregation of chunk results is not performed in chunk mode, so the `aggregator` function is not needed. 
```
        select json_Serialize(
        dbms_hybrid_vector.search(
        json(
        '{ "hybrid_index_name" : "my_hybrid_idx",
        "vector":
        {
        "search_text"   : "prioritize teamwork and leadership experience",
        "search_mode"   : "CHUNK"
        },
        "return":
        {
        "values"        : [ "rowid", "score", "chunk_text", "chunk_id" ],
        "topN"          : 3
        }
        }'
        )
        ) pretty)
        from dual;
```
        

The top 3 chunks with the highest scores are returned along with chunk IDs and chunk texts. You can also see the corresponding `ROWID`s of the `doc_tab` table rows. The highest score (`61`) of Candidate-1 with chunk ID 1 indicates the top match: 
```
        [
        {
        "rowid"      : "AAAR9jAABAAAQeaAAA",
        "score"      : 61,
        "chunk_text" : "Candidate-1: C Master. Optimizes low-level system (i.e. Da
        tabase) performance with C. Strong leadership skills in guiding teams to deliver
        complex projects.",
        "chunk_id"   : "1"
        },
        {
        "rowid"      : "AAAR9jAABAAAQeaAAD",
        "score"      : 56.64,
        "chunk_text" : "Candidate-4: Database Administrator (DBA). Maintains and
        secures enterprise database (Oracle, MySql, SQL Server). Passionate about data i
        ntegrity and optimization. Strong mentor for junior DBA(s).",
        "chunk_id"   : "1"
        },
        {
        "rowid"      : "AAAR9jAABAAAQeaAAB",
        "score"      : 55.75,
        "chunk_text" : "Candidate-2: Full-Stack Developer. Skilled in Database, C, HTM
        L, JavaScript, and Python with experience in building responsive web application
        s. Thrives in collaborative team environments.",
        "chunk_id"   : "1"
        }
        ]
```
        

    5. **Pure keyword in document mode**:

This is a pure text-based keyword query to fetch document-level results. Note that this is equivalent to the traditional `CONTAINS` query using Oracle Text. Here, full-text search retrieves the best documents and then internally computes their keyword scores. The top n documents with the highest keyword scores are returned. To know more about how this works, see [Pure Keyword in Document Mode](understand-hybrid-search.md#GUID-310D2298-90F4-4AFE-AF03-F3B81E55F84C__GUID-03905981-A6E9-4D2C-A0DC-0807A95AA3F3). 
```
        select json_Serialize(
        dbms_hybrid_vector.search(
        json(
        '{ "hybrid_index_name" : "my_hybrid_idx",
        "text":
        {
        "contains"        : "C, Python, Database"
        },
        "return":
        {
        "values"          : [ "rowid", "score" ],
        "topN": 3
        }
        }'
        )
        ) pretty)
        from dual;
```
        

The top 3 documents with the highest scores are returned, where you can see the corresponding `ROWID`s of the `doc_tab` table as well as their keyword scores: 
```
        [
        {
        "rowid" : "AAAR9jAABAAAQeaAAB",
        "score" : 68
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAA",
        "score" : 35
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAD",
        "score" : 2
        }
        ]
```
        

    6. **Keyword and semantic in document mode (common query string)**:

This is a hybrid query that conducts both keyword search and vector search on the data, and then combines the keyword scores and semantic scores to fetch document-level results. 

Here, the same text string (`C, Python, Database`) is used for both: 

       * a keyword query on the document text index by converting the `SEARCH_TEXT` string into a `CONTAINS ACCUM` operator syntax 

       * and a semantic query on the vectorized chunk index by vectorizing the `SEARCH_TEXT` string for a `VECTOR_DISTANCE` query 
```
        select json_Serialize(
        dbms_hybrid_vector.search(
        json(
        '{ "hybrid_index_name" : "my_hybrid_idx",
        "search_text"       : "C, Python, Database",
        "search_fusion"     : "INTERSECT",
        "search_scorer"     : "rsf",
        "vector":
        {
        "search_mode"   : "DOCUMENT",
        "aggregator"    : "MAX"
        },
        "return":
        {
        "values"        : [ "rowid", "score" ],
        "topN"          : 3
        }
        }'
        )
        ) pretty)
        from dual;
```
        

For vector search, this query searches the query vector (created from the `SEARCH_TEXT` string) against the vector index. A vector search operates at the chunk level, where the query first extracts the top candidates of vectorized chunks, aggregates them by document using the `aggregator` (`MAX`) function, produces a combined vector score (according to the `aggregator`), and finally returns the top n documents. 

For scoring, first both search results are added using a `UNION ALL` operation. Then the results are fused using the `SEARCH_FUSION` operator, `INTERSECT` that combines all distinct rows selected by both the searches. The final scoring is computed using the defined `SEARCH_SCORER` algorithm, RSF. Finally, the defined `topN` doc IDs are returned at most. 

To know more about how the scores are calculated, see [Keyword and Semantic in Document Mode](understand-hybrid-search.md#GUID-310D2298-90F4-4AFE-AF03-F3B81E55F84C__GUID-7C48C489-FCB6-4838-918F-BA6C847032F4). 

The top 3 documents with the highest scores are returned, where you can see a list of `ROWID`s from your base table (`doc_tab`) corresponding to the list of best files identified. 
```
        [
        {
        "rowid" : "AAAR9jAABAAAQeaAAB",
        "score" : 69.54
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAA",
        "score" : 62.77
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAD",
        "score" : 62.15
        }
        ]
```
        

    7. **Keyword and semantic in document mode (separate query strings)**:

As compared to the previous keyword and semantic search in document mode (where you used the same `search_text` string for both text search and vector search), here you specify two separate search texts for both the search types. 
```
        select json_Serialize(
        dbms_hybrid_vector.search(
        json(
        '{ "hybrid_index_name" : "my_hybrid_idx",
        "search_fusion"     : "INTERSECT",
        "search_scorer"     : "rsf",
        "vector":
        {
        "search_text"   : "prioritize teamwork and leadership experience",
        "search_mode"   : "DOCUMENT",
        "aggregator"    : "MAX"
        },
        "text":
        {
        "contains"      : "C, Python, Database"
        },
        "return":
        {
        "values"        : [ "rowid", "score" ],
        "topN"          : 3
        }
        }'
        )
        ) pretty)
        from dual;
```
        

In the same manner, the keyword search retrieves a list of doc IDs satisfying your `CONTAINS` text query string (`C, Python, Database`). The vector search retrieves a list of doc IDs satisfying your `SEARCH_TEXT` similarity query string (`prioritize teamwork and leadership experience`). 

For scoring, first both search results are added using a `UNION ALL` operation. Then the results are fused using the `SEARCH_FUSION` operator, `INTERSECT`. The final scoring is computed using the defined `SEARCH_SCORER` algorithm, RSF. Finally, the defined `topN` doc IDs are returned at most. 

To know more about how the scores are calculated, see [Keyword and Semantic in Document Mode](understand-hybrid-search.md#GUID-310D2298-90F4-4AFE-AF03-F3B81E55F84C__GUID-7C48C489-FCB6-4838-918F-BA6C847032F4). 

The top 3 documents with the highest scores are returned, where you can see a list of `ROWID`s from your base table (`doc_tab`) corresponding to the list of best files identified. 
```
        [
        {
        "rowid" : "AAAR9jAABAAAQeaAAA",
        "score" : 58.64
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAB",
        "score" : 56.86
        },
        {
        "rowid" : "AAAR9jAABAAAQeaAAD",
        "score" : 51.67
        }
        ]
```
        

    8. **Keyword and semantic in chunk mode**:

This is a hybrid query that conducts both keyword search and vector search on the data, and then combines the keyword scores and semantic scores to fetch chunk-level results. 
```
        select json_Serialize(
        dbms_hybrid_vector.search(
        json(
        '{ "hybrid_index_name" : "my_hybrid_idx",
        "vector":
        {
        "search_text"   : "prioritize teamwork and leadership experience",
        "search_mode"   : "CHUNK"
        },
        "text":
        {
        "contains"      : "C, Python, Database"
        },
        "return":
        {
        "values"        : [ "rowid", "score", "chunk_text", "chunk_id" ],
        "topN": 3
        }
        }'
        )
        ) pretty)
        from dual;
```
        

The keyword search retrieves a list of doc IDs satisfying your `CONTAINS` text query string (`C, Python, Database`). 

The vector search performs a similarity query with the query vector (created from the `SEARCH_TEXT` string: `prioritize teamwork and leadership experience`) against the vector index of all the chunks of all your documents. It retrieves a list of chunk IDs and associated doc IDs satisfying your `SEARCH_TEXT` similarity query string. 

Aggregation of chunk results is not performed in chunk mode, so the `AGGREGATOR` function is not applicable. 

For scoring, first the text document results are added into the chunk results using the `RIGHT OUTER JOIN` operation on doc IDs. Any text document results that are not in the vector candidate chunks are not returned. Then the results are fused using the `SEARCH_FUSION` operator, `INTERSECT` that combines all distinct rows selected by both the searches. The final scoring is computed using the defined `SEARCH_SCORER` algorithm, RSF. Finally, the defined `topN` chunk IDs are returned at most. 

To know how the scores are calculated, see [Keyword and Semantic in Chunk Mode](understand-hybrid-search.md#GUID-310D2298-90F4-4AFE-AF03-F3B81E55F84C__GUID-CB359FD1-B5B0-484F-BDA9-79723ABEB52B). 

A list of top 3 chunk IDs are returned from the files stored in your base table (`doc_tab`), along with chunk texts and the corresponding `ROWID`s. The highest final score (`58.64`) of Candidate-1 with chunk ID 1 indicates the top match: 
```
        [
        {
        "rowid"      : "AAAR9jAABAAAQeaAAA",
        "score"      : 58.64,
        "chunk_text" : "Candidate-1: C Master. Optimizes low-level system (i.e. Da
        tabase) performance with C. Strong leadership skills in guiding teams to deliver
        complex projects.",
        "chunk_id"   : "1"
        },
        {
        "rowid"      : "AAAR9jAABAAAQeaAAB",
        "score"      : 56.86,
        "chunk_text" : "Candidate-2: Full-Stack Developer. Skilled in Database, C, HTM
        L, JavaScript, and Python with experience in building responsive web application
        s. Thrives in collaborative team environments.",
        "chunk_id"   : "1"
        },
        {
        "rowid"      : "AAAR9jAABAAAQeaAAD",
        "score"      : 51.67,
        "chunk_text" : "Candidate-4: Database Administrator (DBA). Maintains and
        secures enterprise database (Oracle, MySql, SQL Server). Passionate about data i
        ntegrity and optimization. Strong mentor for junior DBA(s).",
        "chunk_id"   : "1"
        }
        ]
```
        




**Related Topics**

  * [Manage Hybrid Vector Indexes](manage-hybrid-vector-indexes.md#GUID-F2493927-23F8-4231-862B-6EDFA5A12299)
  * [SEARCH](search.md#GUID-A386BDB0-35D0-41E1-8F41-49AEBEC13BFC)



**Parent topic:** [Perform Hybrid Search](perform-hybrid-search.md)
