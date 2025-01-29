## V$VECTOR_GRAPH_INDEX {#GUID-B037A491-B8BE-4E0B-B14B-5E505742A737}

This fixed view provides diagnostic information about In-Memory Neighbor Graph vector indexes and is available with Autonomous Database Serverless (ADB-S).

Column Name | Data Type | Description  
---|---|---  
`OWNER` |  `VARCHAR2(129)` |  User name of the vector graph index owner  
`INDEX_NAME` |  `VARCHAR2(129)` |  Name of the vector graph index  
`PARTITION_NAME` |  `VARCHAR2(129)` |  Object partition name (set to `NULL` for non-partitioned objects)   
`INDEX_OBJN` |  `NUMBER` |  Index object number  
`ANCHOR_ADDRESS` |  `RAW(8)` |  The address of the anchor structure of the in-memory neighbor graph index in the vector pool  
`INDEX_GRAPH_TYPE` |  `VARCHAR2(129)` |  Type of the constructed graph of this index. Possible values: <br>* `HNSW`: <br>Hierarchical Navigable Small Worlds. Currently only this graph is supported. 

  
`NUM_LAYERS` |  `NUMBER` |  Number of layers in the constructed graph  
`NUM_VECTORS` |  `NUMBER` |  Number of indexed vectors in the vector graph index  
`SPARSE_LAYER_VECTORS` |  `NUMBER` |  The total number of vectors in the sparse layers. Note that a sparse layer is defined as any layer above the bottom most layer.  
`NUM_NEIGHBORS` |  `NUMBER` |  Maximum number of neighbors a vector can have in the sparse layers  
`EFCONSTRUCTION` |  `NUMBER` |  Maximum number of closest vector candidates considered at each step of the search during insertion  
`TOTAL_EDGES` |  `NUMBER` |  Number of total edges in the constructed graph as of the latest snapshot  
`REF_COUNT` | `NUMBER` |  Number used to track readers of the inmemory neighbor graph index. The inmemory neighbor graph index can only be dropped once the `refcount` is 0\.   
`QUERY_DIST_COUNT` |  `NUMBER` |  Number of distance computations done as part of queries that used the inmemory neighbor graph index  
`CREATION_DIST_COUNT` |  `NUMBER` |  Number of distance computations done as part of the inmemory neighbor graph index creation  
`PRUNED_NEIGHBORS` |  `NUMBER` |  Number of neighbors pruned out during the neighbor selection phase of the inmemory neighbor graph index creation  
`NUM_SNAPSHOTS` |  `NUMBER` |  Number of snapshots currently tracked for the inmemory neighbor graph index  
`MAX_SNAPSHOT` |  `NUMBER` |  The total number of snapshots created so far for the inmemory graph index  
`ALLOCATED_BYTES` |  `NUMBER` |  Total amount of memory allocated to this vector graph index, in bytes  
`USED_BYTES` |  `NUMBER` |  Total amount of memory currently used by the vector graph index, in bytes  
`CON_ID` |  `NUMBER` |  The ID of the container to which the data pertains. Possible values include the following: <br>* `0`: <br>This value is used for rows containing data that pertain to the entire multitenant container database (CDB). This value is also used for rows in non-CDBs. <br>`1`: <br>This value is used for rows containing data that pertain to only the root <br>`n`: <br>Where `n` is the applicable container ID for the rows containing data   
  
**Parent topic:** [Vector Index and Hybrid Vector Index Views](vector-index-and-hybrid-vector-index-views.md)
