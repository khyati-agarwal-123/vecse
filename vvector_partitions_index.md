## V$VECTOR_PARTITIONS_INDEX {#GUID-BF60FDFC-7F24-4863-A978-52F0AB239787}

This fixed view provides diagnostic information about Inverted File Flat indexes and is available with Autonomous Database Serverless (ADB-S).

Column Name | Data Type | Description  
---|---|---  
`OWNER` |  `VARCHAR2(129)` |  User name of the vector partition index owner  
`INDEX_NAME` |  `VARCHAR2(129)` |  Name of the vector partition index  
`PARTITION_NAME` |  `VARCHAR2(129)` |  Object partition name (set to `NULL` for non-partitioned objects)   
`INDEX_OBJN` |  `NUMBER` |  Index object number  
`CENTROIDS_ANCHOR_ADDRESS` |  `RAW(8)` |  The address of the anchor structure hosting the centroids for the in-memory neighbor partitions index in the vector pool. `CENTROIDS_ANCHOR_ADDRESS` is only enabled if in-memory centroids are enabled on the index. Set to `0` if in-memory centroids are unavailable for the index.   
`INDEX_PARTITIONS_TYPE` |  `VARCHAR2(129)` |  The partition type of this neighbor partitions vector index. Possible values include: <br>* `IVF`: <br>Inverted File Index 

  
`NUM_NEIGHBOR_PARTITIONS` |  `NUMBER` |  Number of neighbor partitions created for the neighbor partitions vector index  
`NUM_VECTORS` |  `NUMBER` |  Number of indexed vectors in this vector index  
`MIN_PARTITION_VECTORS_COUNT` |  `NUMBER` |  Number of vectors in the smallest neighbor partition  
`MAX_PARTITION_VECTORS_COUNT` | `NUMBER` |  Number of vectors in the largest neighbor partition  
`INDEX_USED_COUNT` |  `NUMBER` |  Number of times this vector partition index has been used by queries  
`AVG_PARTITIONS_SCANNED` |  `NUMBER` |  Average number of neighbor partitions scanned in order to answer queries. This is an average over the `NEIGHBOR PROBES` query parameter supported for an Inverted File neighbor partitions index   
`CON_ID` |  `NUMBER` |  The ID of the container to which the data pertains. Possible values include the following: <br>* `0`: <br>This value is used for rows containing data that pertain to the entire multitenant container database (CDB). This value is also used for rows in non-CDBs. <br>`1`: <br>This value is used for rows containing data that pertain to only the root <br>`n`: <br>Where `n` is the applicable container ID for the rows containing data   
  
**Parent topic:** [Vector Index and Hybrid Vector Index Views](vector-index-and-hybrid-vector-index-views.md)
