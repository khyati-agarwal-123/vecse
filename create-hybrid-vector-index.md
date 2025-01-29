## CREATE HYBRID VECTOR INDEX {#GUID-22B07106-419E-4C55-B4E1-3FD691E033BC}

Use the `CREATE HYBRID VECTOR INDEX` SQL statement to create a hybrid vector index, which allows you to index and query documents using a combination of full-text search and vector similarity search. 

Purpose

To create a class of specialized Domain Index called a hybrid vector index. 

A **hybrid vector index** is an Oracle Text `SEARCH INDEX` type that combines the existing Oracle Text indexing data structures and vector indexing data structures into one unified structure. It is a single domain index that stores both text fields and vector fields for a document. Both text search and similarity search are performed on tokenized terms and vectors respectively. The two search results are combined and scored to return a unified result set. 

The purpose of a hybrid vector index is to enhance search relevance of an Oracle Text index by allowing users to search by both *vectors* and *keywords*. By integrating traditional keyword-based text search with vector-based similarity search, you can improve the overall search experience and provide users with more accurate information. 

Usage Notes

To create a hybrid vector index, you can provide minimal information such as:

  * table or column on which you want to create the index

  * in-database ONNX embedding model for generating embeddings




For cases where multiple columns or tables need to be indexed together, you can specify the `MULTI_COLUMN_DATASTORE` or `USER_DATASTORE` preference. 

All other indexing parameters are predefined to facilitate the indexing of documents without requiring you to be an expert in any text processing, chunking, or embedding strategies. If required, you can modify the predefined parameters using: 

  * Vector search preferences for the *vector index* part of the index 

  * Text search preferences for the *text index* part of the index 

  * Index maintenance preferences for DML operations on the combined index




For detailed information on the creation process of a hybrid vector index or in general about what hybrid vector indexes are, see [Understand Hybrid Vector Indexes](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-28C18166-43BB-4D2C-B8B3-8127D3485578). 

> **note:** There are some key points to note when creating and using hybrid vector indexes. See [Guidelines and Restrictions for Hybrid Vector Indexes](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-8CFB2031-695D-4C9F-92AF-6628124DE82A). 

Syntax
```
    CREATE HYBRID VECTOR INDEX [schema.]index_name ON
    [schema.]table_name(column_name)
    PARAMETERS ('paramstring')
    [FILTER BY filter_column[, filter_column]...]
    [ORDER BY oby_column[desc|asc][, oby_column[desc|asc]]...]
    [PARALLEL n];
```
    

Here is an example DDL specified with only the minimum required parameters.
```
    CREATE HYBRID VECTOR INDEX my_hybrid_idx on
    doc_table(text_column)
    PARAMETERS('MODEL my_embed_model');
```
    

More comprehensive examples are given at the end of this section.

Let us explore all the required and optional indexing parameters:

[*`schema`*.]*`index_name`*
    

Specify the name of the hybrid vector index to create.

[*`schema`*.]*`table_name`*(*`column_name`*) 
    

Specify the name of the table and column on which you want to create the hybrid vector index. You can create a hybrid vector index on one or more text columns with `VARCHAR2`, `CLOB`, and `BLOB` data types. 

> **note:** You cannot create hybrid vector indexes on a text column that uses the `IS JSON` check constraint. 

Because the system can index most document formats, including HTML, PDF, Microsoft Word, and plain text, you can load a supported type into the text column. For a complete list, see [Supported Document Formats](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-BDD5A962-25F7-43B1-B7C3-37B3A467AC0A). 

For cases where multiple columns or tables need to be indexed together, specify a datastore preference (described later in Text search preferences).

PARAMETERS (paramstring) 
    

Specify preferences in `paramstring`: 

  * **Vector Search Preferences**: 

Configures the "vector index" part of a hybrid vector index, pertaining to processing input for vector search. 

> **note:** You can either pass a minimal set of parameters (the required `MODEL` and the optional `VECTOR_IDXTYPE` parameters) directly in the `PARAMETERS` clause or use a vectorizer preference to specify a complete set of vector search parameters. You cannot use both (directly set parameters along with vectorizer) in the `PARAMETERS` clause. 

    * With `MODEL` and `VECTOR_IDXTYPE` directly specified: 
```
        CREATE HYBRID VECTOR INDEX [schema.]index_name ON
        [schema.]table_name(column_name)
        PARAMETERS ('**MODEL**
        [**VECTOR_IDXTYPE** ]')
        [FILTER BY filter_column[, filter_column]...]
        [ORDER BY oby_column[desc|asc][, oby_column[desc|asc]]...]
        [PARALLEL n];
```
        

Here, `MODEL` specifies the vector embedding model that you import into the database for generating vector embeddings on your input data. 

> **note:** Currently, only ONNX in-database embedding models are supported. 

`VECTOR_IDXTYPE` specifies the type of vector index to create, such as `IVF` (default) for the Inverted File Flat (IVF) vector index and `HNSW` for the Hierarchical Navigable Small World (HNSW) vector index. If you omit this parameter, then the IVF vector index is created by default. 

Creating a `LOCAL` index on an Hybrid Vector Index is supported when the underlying `index_type` is `IVF`. An example is shown below: 
```
        CREATE HYBRID VECTOR INDEX my_hybrid_idx on
        doc_table(text_column)
        parameters('MODEL my_doc_model
        **VECTOR_IDXTYPE IVF**')
        LOCAL PARALLEL;
```
        

> **note:** Caution:Creating a `LOCAL` index on Hybrid Vector Index when the underlying `index_type` is `HNSW`, would throw an error before starting any document processing (early failure). 

    * With the vectorizer preference:

A vectorizer preference is a JSON object that collectively holds all indexing parameters related to chunking (`UTL_TO_CHUNKS` or `VECTOR_CHUNKS`), embedding (`UTL_TO_EMBEDDING`, `UTL_TO_EMBEDDINGS`, or `VECTOR_EMBEDDING`), and vector index (`distance`, `accuracy`, or `vector_idxtype`). 

You use the `DBMS_VECTOR_CHAIN.CREATE_PREFERENCE` PL/SQL function to create a vectorizer preference. To create a vectorizer preference, see [DBMS_VECTOR_CHAIN.CREATE_PREFERENCE](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-B83978CD-EAF8-4794-9652-F335C54C3385). 

After creating a vectorizer preference, you can use the `VECTORIZER` parameter to pass the preference name here. For example: 
```
        begin
        DBMS_VECTOR_CHAIN.CREATE_PREFERENCE(
        'my_vectorizer_pref',
        dbms_vector_chain.vectorizer,
        json('{
        "vector_idxtype":  "hnsw",
        "model"         :  "my_doc_model",
        "by"            :  "words",
        "max"           :  100,
        "overlap"       :  10,
        "split"         :  "recursively"
        }'
        ));
        end;
        /
        
        CREATE HYBRID VECTOR INDEX my_hybrid_idx on
        doc_table(text_column)
        parameters('VECTORIZER my_vectorizer_pref');
```
        

  * **Text Search Preferences**: 

Configures the "Oracle Text index" part of a hybrid vector index, pertaining to processing input for keyword search. 

These parameters define the text processing and tokenization stages of a hybrid vector indexing pipeline. All these are the same set of parameters that you provide when working with Oracle Text indexes alone.
```
    [DATASTORE datastore_pref]
    [STORAGE storage_pref]
    [MEMORY memsize]
    [STOPLIST stoplist]
    [LEXER lexer_pref]
    [FILTER filter_pref]
    [WORDLIST wordlist_pref]
    [SECTION GROUP section_group]
```
    

DATASTORE datastore_pref 
    

Specify the name of your datastore preference. Use the datastore preference to specify the local or remote location where your source files are stored. 

If you want to index multiple columns or tables together, see [MULTI_COLUMN_DATASTORE](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-09F6DE00-B354-443E-9174-71CE88F5942E) and [USER_DATASTORE](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-2EE366EC-0748-48D5-81BF-7D6430371358). 

For a complete list of all datastore preferences, see [Datastore Types](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-91576377-DD67-4CBA-91AD-127402C4B93A). 

Default: `DIRECT_DATASTORE`

STORAGE storage_pref 
    

Specify the name of your storage preference for an Oracle Text search index. Use the storage preference to specify how the index tables are stored. See [Storage Types](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-C6DA9061-955A-403E-B8BB-961D31360DCB). 

MEMORY memsize 
    

Specify the amount of run-time memory to use for indexing.
```
    memsize = number[K|M|G]
```
    

K is for kilobytes, M is for megabytes, and G is for gigabytes.

The value you specify for `memsize` must be between 1M and the value of `MAX_INDEX_MEMORY` in the `CTX_PARAMETERS` view. To specify a memory size larger than the `MAX_INDEX_MEMORY`, you must reset this parameter with `CTX_ADM.SET_PARAMETER` to be larger than or equal to `memsize`. See [CTX_ADM.SET_PARAMETER](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-34F6AE6C-B3B3-4A08-87EA-A75849D4ED30). 

The default for Oracle Text search index is 500 MB.

The `memsize` parameter specifies the amount of memory Oracle Text uses for indexing before flushing the index to disk. Specifying a large amount memory improves indexing performance because there are fewer I/O operations and improves query performance and maintenance, because there is less fragmentation. 

Specifying smaller amounts of memory increases disk I/O and index fragmentation, but might be useful when run-time memory is scarce.

STOPLIST stoplist 
    

Specify the name of your stoplist. Use stoplist to identify words that are not to be indexed. See [CTX_DDL.CREATE_STOPLIST](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-3336B8E9-13FB-4997-A9AD-8D9A207B10C4). 

Default: `CTXSYS.DEFAULT_STOPLIST`

LEXER lexer_pref 
    

Specify the name of your lexer or multilexer preference. Use the lexer preference to identify the language of your text and how text is tokenized for indexing. See [Lexer Types](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-5CE1AEE2-92A9-4BE9-B6A7-3E4519FAB0D1). 

Default: `CTXSYS.DEFAULT_LEXER`

FILTER filter_pref 
    

Specify the name of your filter preference. Use the filter preference to specify how to filter formatted documents to plain text. See [Filter Types](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-24A7DF95-CDA5-4062-82D5-6DFC45F736BF). 

The default for binary text columns is `NULL_FILTER`. The default for other text columns is `AUTO_FILTER`. 

WORDLIST wordlist_pref 
    

Specify the name of your wordlist preference. Use the wordlist preference to enable features such as fuzzy, stemming, and prefix indexing for better wildcard searching. See [Wordlist Type](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-9CE95823-DF47-43E4-940A-9B1E8CB5E21F). 

SECTION GROUP section_group 
    

Specify the name of your section group. Use section groups to create sections in structured documents. See [CTX_DDL.CREATE_SECTION_GROUP](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-9B0E0A60-53DF-4B90-827A-8D1B8D313A16). 

Default: `NULL_SECTION_GROUP`

  * **Index Maintenance Preferences**: 

Configures the DML operations on the entire hybrid vector index, that is, how to synchronize and optimize the index. 

Because a hybrid vector index is basically an Oracle Text search index type, so all maintenance-specific capabilities of an Oracle Text index are applicable.
```
    [MAINTENANCE AUTO | MAINTENANCE MANUAL]
    [SYNC (MANUAL | EVERY "interval-string" | ON COMMIT)]
    [OPTIMIZE (MANUAL | AUTO_DAILY | EVERY "interval-string")]
```
    

MAINTENANCE AUTO | MAINTENANCE MANUAL 
    

Specify the maintenance type for synchronization of a hybrid vector index when there are inserts, updates, or deletes to the base table. The maintenance type specified for an index applies to all index partitions.

You can specify one of the following maintenance types:

Maintenance Type | Description  
---|---  
`MAINTENANCE AUTO` |  This method sets your index to automatic maintenance, that is, the index is automatically synchronized in the background at optimal intervals. <br>You do not need to manually configure a `SYNC` type or set any synchronization interval. The background mechanism automatically determines the synchronization interval and schedules background `SYNC.INDEX` operations by tracking the DML queue.   
`MAINTENANCE MANUAL` |  This method sets your index to manual maintenance. This is a non-automatic maintenance (synchronization) mode in which you can specify `SYNC` types, such as `MANUAL`, `EVERY`, or `ON COMMIT`.   
  
SYNC (MANUAL | EVERY "interval-string" | ON COMMIT) 
    

Specify the `SYNC` type for synchronization of a hybrid vector index when there are inserts, updates, or deletes to the base table. These `SYNC` settings are applicable only to the indexes that are set to manual maintenance. 

> **note:** By default, a hybrid vector index runs in an automatic maintenance mode (`MAINTENANCE AUTO`), which means that your DMLs are automatically synchronized into the index in the background at optimal intervals. Therefore, you do not need to manually configure a `SYNC` type for maintaining a hybrid vector index. However, if required, you can do so if you want to modify the default settings for an index. 

You can specify one of the `SYNC` methods: 

SYNC Type | Description  
---|---  
`MANUAL` |  With this method, automatic synchronization is not provided. You must manually synchronize the index using `CTX_DDL.SYNC_INDEX`.   
`EVERY` *`interval-string`* |  Automatically synchronize the index at a regular interval specified by the value of *`interval-string`*, which takes the same syntax as that for scheduler jobs. Automatic synchronization using `EVERY` requires that the index creator have `CREATE` `JOB` privileges. <br>Ensure that *`interval-string`* is set to a considerable time period so that any previous synchronization jobs will have completed. Otherwise, the synchronization job may stop responding. The *`interval-string`* argument must be enclosed in double quotation marks ('' '').   
`ON` `COMMIT` |  Synchronize the index immediately after a commit. The commit does not return until the sync is complete.<br>The operation uses the memory specified with the *`memory`* parameter. <br>This sync type works best when the `STAGE_ITAB` index option is enabled, otherwise it causes significant fragmentation of the main index, requiring frequent `OPTIMIZE` calls.   
  
With automatic (`EVERY`) synchronization, you can specify memory size and parallel synchronization. You can define repeating schedules in the *`interval-string`* argument using calendaring syntax values. These values are described in [*Oracle Database PL/SQL Packages and Types Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-73622B78-EFF4-4D06-92F5-E358AB2D58F3). 

Syntax:
```
    SYNC [EVERY "interval-string"] MEMORY mem_size PARALLEL paradegree
```
    

For example, to sync the index at an interval of 20 seconds:
```
    SYNC [EVERY "freq=secondly;interval=20"] MEMORY 500M PARALLEL 2
```
    

OPTIMIZE (MANUAL | AUTO_DAILY | EVERY "interval-string) 
    

Specify `OPTIMIZE` to enable automatic background index optimization of a hybrid vector index. You can specify any one of the following `OPTIMIZE` methods: 

OPTIMIZE Type | Description  
---|---  
`MANUAL` |  Provides no automatic optimization. You must manually optimize the index with `CTX_DDL.OPTIMIZE_INDEX`.   
`AUTO_DAILY` |  This is the default setting. With `OPTIMIZE (AUTO_DAILY)`, the optimize `FULL` job is scheduled to run midnight from 12 A.M. local time everyday.   
`EVERY "*`interval-string`*"` |  Automatically runs optimize `token` at a regular interval specified by the value *`interval-string`*, which takes the same syntax as the scheduler jobs. <br>Ensure that *`interval-string`* is set to a considerable time period so that the previous optimize jobs are complete; otherwise, the optimize job might stop responding. *`interval-string`* must be enclosed in double quotes, and any single quote within *`interval-string`* must be preceded by the escape character with another single quote.   
  
With `AUTO_DAILY | EVERY "*`interval-string`*"` setting, you can specify parallel optimization. 

Syntax:
```
    OPTIMIZE [AUTO_DAILY | EVERY "interval-string"] PARALLEL paradegree ...
```
    

For example, to optimize the index at an interval of 20 minutes:
```
    OPTIMIZE [EVERY "freq=minutely;interval=20"] PARALLEL 2
```
    




FILTER BY filter_column 
    

Specify the structured indexed column on which a range or equality predicate in the `WHERE` clause of a mixed query will operate. You can specify one or more structured columns for `filter_column`, on which the relational predicates are expected to be specified along with the `CONTAINS()` predicate in a query. 

  * You can use these relational operators: 

`<`, `<=`, `=`, `>=`, `>`, `between`, and `LIKE` (for `VARCHAR2`) 

  * These columns can only be of `CHAR`, `NUMBER`, `DATE`, `VARCHAR2`, or `RAW` type. Additionally, `CHAR`, `VARCHAR2` and `VARCHAR2` types are supported only if the maximum length is specified and does not exceed 249 bytes. 

If the maximum length of a `CHAR` or `VARCHAR2` column is specified in characters, for example, `VARCHAR2` (50 `CHAR`), then it cannot exceed `FLOOR` (249/`max_char_width`), where `max_char_width` is the maximum width of any character in the database character set. 

For example, the maximum specified column length cannot exceed 62 characters, if the database character set is AL32UTF8. The `ADT` attributes of supported types (`CHAR`, `NUMBER`, `DATE`, `VARCHAR2`, or `RAW`) are also allowed. 

An error is raised for all other data types. Expressions, for example, `func(cola)`, and virtual columns are not allowed. 

  * `txt_column` is allowed in the `FILTER` `BY` column list. 

  * DML operations on `FILTER` `BY` columns are always transactional. 




ORDER BY oby_column[desc|asc] 
    

Specify one or more structured indexed columns by which you want to sort query results.

You can specify a list of structured *`oby_columns`*. These columns can only be of `CHAR`, `NUMBER`, `DATE`, `VARCHAR2`, or `RAW` type. `VARCHAR2` and `RAW` columns longer than `249` bytes are truncated to the first `249` bytes. Expressions, for example `func(cola)`, and virtual columns are not allowed. 

The order of the specified columns matters. The `ORDER BY` clause in a query can contain: 

  * The entire ordered `ORDER BY` columns 

  * Only the prefix of the ordered `ORDER BY` columns 

  * The score followed by the prefix of the ordered `ORDER BY` columns 




`DESC` sorts the results in a descending order (from highest to lowest), while `ASC` (default) sorts the results in an ascending order (from lowest to highest). 

[PARALLEL n] 
    

Parallel indexing can improve index performance when you have multiple CPUs. To create an index in parallel, use the `PARALLEL` clause with a parallel degree. 

Optionally specifies the parallel degree for parallel indexing. The actual degree of parallelism might be smaller depending on your resources. You can use this parameter on nonpartitioned tables. However, creating a nonpartitioned index in parallel does not turn on parallel query processing. Parallel indexing is supported for creating a local partitioned index.

The indexing memory size specified in the parameter clause applies to each parallel worker. For example, if indexing memory size is specified in the parameter clause as 500M and parallel degree is specified as 2, then you must ensure that there is at least 1GB of memory available for indexing. 

Examples

  * **With vector search preferences directly specified**: 

In this example, only the required parameter `model` is specified in the `PARAMETERS` clause: 
```
    CREATE HYBRID VECTOR INDEX my_hybrid_idx on
    doc_table(text_column)
    parameters('**MODEL** my_doc_model');
```
    

In this example, both the parameters `model` and `vector_idxtype` are specified: 
```
    CREATE HYBRID VECTOR INDEX my_hybrid_idx on
    doc_table(text_column)
    parameters('**MODEL** my_doc_model
    **VECTOR_IDXTYPE** HNSW');
```
    

  * **With vector search preferences specified using VECTORIZER**: 

In this example, the `vectorizer` parameter is used in the `PARAMETERS` clause to specify the `my_vectorizer_spec` preference: 
```
    begin
    DBMS_VECTOR_CHAIN.CREATE_PREFERENCE(
    'my_vectorizer_spec',
    dbms_vector_chain.vectorizer,
    json('{"vector_idxtype" :  "hnsw",
    "model"          :  "my_doc_model",
    "by"             :  "words",
    "max"            :  100,
    "overlap"        :  10,
    "split"          :  "recursively"}'));
    end;
    /
    
    CREATE HYBRID VECTOR INDEX my_hybrid_idx on
    doc_table(text_column)
    parameters('**VECTORIZER** my_vectorizer_spec');
```
    

  * **With text search and vector search preferences directly specified**: 

In this example, only the required Vector Search parameter `MODEL` is specified in the `PARAMETERS` clause. Text Search parameters are also specified: 
```
    CREATE HYBRID VECTOR INDEX my_hybrid_idx on
    doc_table(text_column)
    parameters('MODEL my_doc_model
    DATASTORE my_datastore
    STORAGE my_storage
    STOPLIST my_stoplist
    LEXER my_lexer')
    ORDER BY docid asc;
```
    

  * **With text search and index maintenance preferences directly specified and vector search preferences specified using VECTORIZER**: 

In this example, the `VECTORIZER` parameter is used to specify the `my_vectorizer_spec` preference that holds vector search parameters. All the Text Search and Index Maintenance preferences are directly specified. 
```
    begin
    DBMS_VECTOR_CHAIN.CREATE_PREFERENCE(
    'my_vectorizer_spec',
    dbms_vector_chain.vectorizer,
    json('{
    "vector_idxtype" :  "hnsw",
    "model"          :  "my_doc_model",
    "by"             :  "words",
    "max"            :  100,
    "overlap"        :  10,
    "split"          :  "recursively"
    }'
    ));
    end;
    /
    
    CREATE HYBRID VECTOR INDEX my_hybrid_idx on
    doc_table(text_column)
    parameters('VECTORIZER my_vectorizer_spec
    DATASTORE my_datastore
    STORAGE my_storage
    MEMORY 128M
    MAINTENANCE AUTO
    OPTIMIZE AUTO_DAILY
    STOPLIST my_stoplist
    LEXER my_lexer
    FILTER my_filter
    WORDLIST my_wordlist
    SECTION GROUP my_section_group')
    FILTER BY category, author
    ORDER BY score(10), score(20) desc
    PARALLEL 3;
```
    




**Related Topics**

  * [Perform Hybrid Search](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-531F9E71-2CEF-46A8-AA53-7C161E801E3A)
  * [Query Hybrid Vector Indexes End-to-End Example](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-4E340969-349E-4E46-AD98-AC2A0AEDB773)



**Parent topic:** [Manage Hybrid Vector Indexes](manage-hybrid-vector-indexes.md)
