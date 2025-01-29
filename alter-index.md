## ALTER INDEX {#GUID-E3AE9E1F-5FA1-40A2-9D46-865058EB24A5}

Use the `ALTER INDEX` SQL statement to modify an existing hybrid vector index. 

Purpose

To make changes to hybrid vector indexes.

Syntax
```
    ALTER INDEX [schema.]index_name REBUILD
    PARAMETERS(
    ['UPDATE VECTOR INDEX']
    ['REPLACE vectorizer vectorizer_pref_name']
    )
    [PARALLEL n];
```
    

> **note:** 

  * If you do not specify the `PARAMETERS` clause, then all parts of the hybrid vector index (both Oracle Text index and vector index) are recreated with existing preference settings. 

  * Renaming hybrid vector indexes using the `ALTER INDEX RENAME` syntax is not supported. 

  * The `ALTER INDEX` parameter `UDPATE VECTOR INDEX` is not supported for Local HVI and HNSW vector indexes. 




[*`schema`*.]*`index_name`*
    

Specifies name of the hybrid vector index that you want to modify.

PARAMETERS(UPDATE VECTOR INDEX) 
    

Recreates only the vector index part of a hybrid vector index with the original preference settings.

PARAMETERS(REPLACE vectorizer *`vectorizer_pref_name`*) 
    

Recreates only the vector index part of a hybrid vector index with the specified vectorizer preference settings. 

> **note:** For non HVI index, replace operation would throw an error, as it cannot replace something that was not present. 

PARALLEL 
    

Specifies parallel indexing, as described for the `CREATE HYBRID VECTOR INDEX` statement. 

For detailed information on the `PARALLEL` clause, see [CREATE HYBRID VECTOR INDEX](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-22B07106-419E-4C55-B4E1-3FD691E033BC). 

Examples

Here are some examples on how you can modify existing hybrid vector indexes: 

  * **To rebuild all parts of a hybrid vector index**: 

Use the following syntax to rebuild all parts of a hybrid vector index (both Oracle Text index and vector index) with the original preference settings:

Syntax:
```
    ALTER INDEX index_name REBUILD [PARALLEL n];
```
    

Note that you do not need to specify any `PARAMETERS` clause when rebuilding both parts of a hybrid vector index. 

Example:
```
    ALTER INDEX **my_hybrid_idx** REBUILD;
    
    SELECT (select id from doc_table where rowid = jt.doc_rowid) as doc,
    jt.chunk
    FROM JSON_TABLE(
    DBMS_HYBRID_VECTOR.SEARCH(
    json(
    '{ "hybrid_index_name" : "my_hybrid_idx",
    "vector" :
    { "search_text"    : "vector based search capabilities",
    "search_mode"    : "CHUNK"
    },
    "return" :
    { "topN"           : 10 }
    }')
    ),
    '$[*]' COLUMNS doc_rowid  PATH '$.rowid',
    chunk      PATH '$.chunk_text') jt;
```
    

  * **To rebuild only the vector index part**: 

Use the following syntax to rebuild only the vector index part of a hybrid vector index with the original preference settings:

Syntax:
```
    ALTER INDEX index_name REBUILD
    PARAMETERS('UPDATE VECTOR INDEX') [PARALLEL n];
```
    

Example:
```
    ALTER INDEX **my_hybrid_idx** REBUILD
    PARAMETERS('**UPDATE VECTOR INDEX**') PARALLEL 3;
    
    SELECT (select id from doc_table where rowid = jt.doc_rowid) as doc,
    jt.chunk
    FROM JSON_TABLE(
    DBMS_HYBRID_VECTOR.SEARCH(
    json(
    '{ "hybrid_index_name" : "my_hybrid_idx",
    "vector" :
    { "search_text"    : "vector based search capabilities",
    "search_mode"    : "CHUNK"
    },
    "return" :
    { "topN"           : 10 }
    }')
    ),
    '$[*]' COLUMNS doc_rowid  PATH '$.rowid',
    chunk      PATH '$.chunk_text') jt;
```
    

  * **To recreate indexes with a vectorizer preference**: 

You can create a vectorizer preference using the `DBMS_VECTOR_CHAIN.CREATE_PREFERENCE` PL/SQL function. For detailed information on how to create a vectorizer preference, see [CREATE_PREFERENCE](create_preference.md#GUID-B83978CD-EAF8-4794-9652-F335C54C3385). After creating the preference, use the `REPLACE vectorizer` parameter to pass the preference name here. 

Syntax:
```
    ALTER INDEX index_name REBUILD
    parameters('REPLACE vectorizer vectorizer_pref_name') [PARALLEL n];
```
    

> **note:** For non-HVI index, the `REPLACE` operation would throw an error. 

Example:
```
    ALTER INDEX **my_hybrid_idx** REBUILD
    parameters('**REPLACE vectorizer** my_vectorizer_pref') [PARALLEL n];
```
    

  * **To replace only the model and/or vector index type**

For an existing HVI index, you can replace the model and/or the index type without specifying the full vectorizer preference using the following syntax.

Syntax:
```
    ALTER INDEX schema.index_name REBUILD[
    parameters('REPLACE MODEL model_name VECTOR_IDXTYPE hnsw/ivf')];
```
    

Example:
```
    ALTER INDEX schema.my_hybrid_idx REBUILD[
    parameters('**REPLACE MODEL** my_model_name **VECTOR_IDXTYPE** ivf')];
```
    

> **note:** For non HVI indexes, this syntax would throw an error. If a vectorizer is also specified alongside the model and/or vector_idxtype, it would lead to an error, as only one of either vectorizer or model/vector_idxtype is allowed. 

  * **Converting an existing Text SEARCH or JSON SEARCH index to an HVI**

You can convert (up-convert) an existing Text SEARCH or JSON SEARCH index to an HVI without a full rebuild of the textual/JSON parts of the index using the following syntax. This syntax is supported only for non-HVI index. After this operation, the index would be modified to become an HVI index. This would mean that all HVI operation would be supported and ctx_report would reflect this update correctly.

Syntax:
```
    ALTER INDEX schema.index_name REBUILD
    [parameters('add vectorizer_fast vectorizer_pref_name')];
```
    

Example:
```
    ALTER INDEX schema.my_hybrid_idx REBUILD
    [parameters('**add vectorizer_fast** my_vectorizer_pref')];
```
    

For other cases that are inapplicable to "up-convert", you can convert an existing Text SEARCH or JSON SEARCH index to an HVI with a full rebuild using the following syntax :
```
    ALTER INDEX schema.index_name REBUILD
    [parameters('add vectorizer vectorizer_pref_name')];
```
    

Example: 
```
    ALTER INDEX schema.my_hybrid_idx REBUILD
    [parameters('**add vectorizer** my_vectorizer_pref)'];
```
    




**Related Topics**

  * [*Oracle Text Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-47E60252-C731-46A8-B587-AE30C1634F48)



**Parent topic:** [Manage Hybrid Vector Indexes](manage-hybrid-vector-indexes.md)
