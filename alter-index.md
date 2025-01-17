## ALTER INDEX {#GUID-E3AE9E1F-5FA1-40A2-9D46-865058EB24A5}

Use the `ALTER INDEX` SQL statement to modify an existing hybrid vector index. 

Purpose<br>To make changes to hybrid vector indexes.

Syntax
```
ALTER INDEX [schema.]index_name REBUILD
[PARAMETERS('UPDATE VECTOR INDEX')]
[PARALLEL n];
```


> **note:** 

* If you do not specify the `PARAMETERS` clause, then all parts of the hybrid vector index (both Oracle Text index and vector index) are recreated with existing preference settings. 

* Renaming hybrid vector indexes using the `ALTER INDEX RENAME` syntax is not supported. 




[*`schema`*.]*`index_name`*


Specifies name of the hybrid vector index that you want to modify.

PARAMETERS(UPDATE VECTOR INDEX) 


Recreates only the vector index part of a hybrid vector index with the original preference settings.

PARALLEL 


Specifies parallel indexing, as described for the `CREATE HYBRID VECTOR INDEX` statement. 

For detailed information on the `PARALLEL` clause, see [CREATE HYBRID VECTOR INDEX](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-22B07106-419E-4C55-B4E1-3FD691E033BC). 

Examples

Here are some examples on how you can modify existing hybrid vector indexes: 

* **To rebuild all parts of a hybrid vector index**: <br>Use the following syntax to rebuild all parts of a hybrid vector index (both Oracle Text index and vector index) with the original preference settings:<br>Syntax:
```
ALTER INDEX index_name REBUILD [PARALLEL n];
```


Note that you do not need to specify any `PARAMETERS` clause when rebuilding both parts of a hybrid vector index. <br>Example:
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


* **To rebuild only the vector index part**: <br>Use the following syntax to rebuild only the vector index part of a hybrid vector index with the original preference settings:<br>Syntax:
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





**Related Topics**

* [*Oracle Text Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-47E60252-C731-46A8-B587-AE30C1634F48)



**Parent topic:** [Manage Hybrid Vector Indexes](manage-hybrid-vector-indexes.md)