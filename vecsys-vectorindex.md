## VECSYS.VECTOR$INDEX {#GUID-FA5183EA-3C60-470D-9C91-8215BE4FF138}

This dictionary table contains detailed information about vector indexes.

Column Name | Data Type | Description  
---|---|---  
`IDX_OBJN` |  `NUMBER` |  Object number of the vector index  
`IDX_OBJD` |  `NUMBER` |  ID of the vector index object. This ID can be used to rebuild the vector index.  
`IDX_OWNER#` |  `NUMBER` |  Owner ID of the vector index. Refer user$ entry  
`IDX_NAME` |  `VARCHAR2(128)` |  Name of the vector index.  
`IDX_BASE_TABLE_OBJN` |  `NUMBER` |  Base table object number  
`IDX_PARAMS` |  `JSON` |  Vector index creation parameters such as vector column indexed, index distance, vector dimension datatype, number of dimensions, efConstruction, and number of neighbors for in-memory neighbor graph HNSW index or the number of centroids for Inverted File Flat vector indexes.  
`IDX_AUXILIARY_TABLES` |  `JSON` |  Names and object IDs of auxiliary tables used to support rowid-to-vertexid conversion information or names of a centroid table and its associated partitions table for Inverted File Flat vector indexes.  
  
Example
```
    SQL> SELECT JSON_SERIALIZE(IDX_PARAMS returning varchar2 PRETTY)
    2* FROM VECSYS.VECTOR$INDEX where IDX_NAME = 'DOCS_HNSW_IDX';
    
    JSON_SERIALIZE(IDX_PARAMSRETURNINGVARCHAR2PRETTY)
    {
    "type" : "HNSW",
    "num_neighbors" : 32,
    "efConstruction" : 300,
    "upcast_dtype" : 1,
    "distance" : "COSINE",
    "accuracy" : 95,
    "vector_type" : "FLOAT32",
    "vector_dimension" : 384,
    "degree_of_parallelism" : 1,
    "indexed_col" : "EMBED_VECTOR"
    }
    SQL>
    SQL> select * from vecsys.vector$index;
    
    IDX_OBJN IDX_OBJD  IDX_OWNER# IDX_NAME       IDX_BASE_TABLE_OBJN  IDX_PARAMS                             IDX_AUXILIARY_TABLES                                                                                    IDX_SPARE1 IDX_SPARE2
    -------- --------  ---------- -------------- -------------------  --------------------------             --------------------                                                                                    ----------  ---------
    74051     143               DOCS_HNSW_IDX              73497   {"type":"HNSW",                        {"rowid_vid_map_objn":74052,
    "num_neighbors":32,                    "shared_journal_transaction_commits_objn":74054,
    "efConstruction":300,                  "shared_journal_change_log_objn":74057,
    "upcast_dtype":1,                      "rowid_vid_map_name":"VECTOR$DOCS_HNSW_IDX$HNSW_ROWID_VID_MAP",
    "distance":"COSINE",                   "shared_journal_transaction_commits_name":"VECTOR$DOCS_HNSW_IDX$HNSW_SHARED_JOURNAL_TRANSACTION_COMMITS",
    "accuracy":95,                         "shared_journal_change_log_name":"VECTOR$DOCS_HNSW_IDX$HNSW_SHARED_JOURNAL_CHANGE_LOG"}
    "vector_type":"FLOAT32",
    "vector_dimension":384,
    "degree_of_parallelism":1,
    "indexed_col":"EMBED_VECTOR"}
    
    74072      143             GALAXIES_HNSW_IDX           74069   {"type":"HNSW",                         {"rowid_vid_map_objn":74073,
    "num_neighbors":32,                     "shared_journal_transaction_commits_objn":74075,
    "efConstruction":300,                   "shared_journal_change_log_objn":74078,
    "upcast_dtype":0,                       "rowid_vid_map_name":"VECTOR$GALAXIES_HNSW_IDX$HNSW_ROWID_VID_MAP",
    "distance":"COSINE",                    "shared_journal_transaction_commits_name":"VECTOR$GALAXIES_HNSW_IDX$HNSW_SHARED_JOURNAL_TRANSACTION_COMMITS",
    "accuracy":95,                          "shared_journal_change_log_name":"VECTOR$GALAXIES_HNSW_IDX$HNSW_SHARED_JOURNAL_CHANGE_LOG"}
    "vector_type":"INT8",
    "vector_dimension":5,
    "degree_of_parallelism":1,
    "indexed_col":"EMBEDDING"}
    
    
    SQL>
```
    

**Parent topic:** [Vector Index and Hybrid Vector Index Views](vector-index-and-hybrid-vector-index-views.md)
