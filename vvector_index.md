## V$VECTOR_INDEX {#GUID-183E165B-9155-41F7-9FE2-160D6BF73849}

This fixed view provides diagnostic information about vector indexes and is available with Autonomous Database Serverless (ADB-S).

Column Name | Data Type | Description  
---|---|---  
`OWNER` |  `VARCHAR(129)` |  User name of the vector index owner  
`INDEX_NAME` |  `VARCHAR(129)` |  Name of the vector index  
`PARTITION_NAME` |  `VARCHAR(129)` |  Object partition name (set to `NULL` for non-partitioned objects)   
`INDEX_ORGANIZATION` |  `VARCHAR(129)` |  The vector index organization. The value can be one of the following: <br>* `NEIGHBOR PARTITIONS`<br>`INMEMORY NEIGHBOR GRAPH`

  
`INDEX_OBJN` |  `NUMBER` |  Index object number  
`ALLOCATED_BYTES` |  `NUMBER` |  Total amount of memory allocated to this vector index, in bytes.<br>When the vector index is an IVF index, the value is `0`.   
`USED_BYTES` |  `NUMBER` |  Total amount of memory currently used by the vector index, in bytes.<br>When the vector index is an IVF index, the value is `0`.   
`NUM_VECTORS` |  `NUMBER` |  The number of vectors currently indexed in the main-memory storage of the index as of the latest snapshot  
`NUM_REPOP` |  `NUMBER` |  Number of times this index has been repopulated  
`INDEX_USED_COUNT` |  `NUMBER` |  Number of times this vector index has been used by queries  
`DISTANCE_TYPE` |  `VARCHAR2(129)` |  Possible distance values: <br>* `EUCLIDEAN`<br>`EUCLIDEAN_SQUARED`<br>`COSINE`<br>`DOT`<br>`MANHATTAN`<br>`HAMMING`

  
`INDEX_DIMENSIONS` |  `NUMBER` |  Number of dimensions of the indexed vector  
`INDEX_DIM_TYPE` |  `VARCHAR2(129)` |  Type of the dimensions of the index vector. Possible values: <br>* `FLOAT16`<br>`FLOAT32`<br>`FLOAT64`<br>`INT8`

  
`DEFAULT_ACCURACY` |  `NUMBER` |  The accuracy to achieve when performing approximate search on this vector index if a target query accuracy is not provided  
`CON_ID` |  `NUMBER` |  The ID of the container to which the data pertains. Possible values include the following: <br>* `0`: <br>This value is used for rows containing data that pertain to the entire multitenant container database (CDB). This value is also used for rows in non-CDBs. <br>`1`: <br>This value is used for rows containing data that pertain to only the root. <br>`n`: <br>Where `n` is the applicable container ID for the rows containing data. 

  
  
**Parent topic:** [Vector Index and Hybrid Vector Index Views](vector-index-and-hybrid-vector-index-views.md)
