## V$VECTOR_MEMORY_POOL {#GUID-9F478477-C486-432E-A473-DF29037AB3CE}

This view contains information about the space allocation for Vector Memory.

The Vector Memory Pool area is used primarily to maintain in-memory vector indexes or metadata useful for vector-related operations. The Vector Memory Pool is subdivided into two pools: a 1MB pool used to store In-Memory Neighbor Graph Index allocations; and a 64K pool used to store metadata. The amount of available memory in each pool is visible in the `V$VECTOR_MEMORY_POOL` view. The relative size of the two pools is determined by internal heuristics. The size of the Vector Memory Pool is controlled by the vector_memory_size parameter. Area in the Vector Memory Pool is also allocated to accelerate Neighbor Partition Index access by storing centroid vectors. 

Column Name | Data Type | Description  
---|---|---  
`POOL` |  `VARCHAR2(26)` |  Name of the pools in the Vector Memory Pool (64K or 1MB)  
`ALLOC_BYTES` |  `NUMBER` |  Total amount of memory allocated to this pool  
`USED_BYTES` |  `NUMBER` |  Amount of memory currently used in this pool  
`POPULATE_STATUS` |  `VARCHAR2(26)` |  Status of the vector memory store—whether it is being populated, is done populating etc.  
`CON_ID` |  `NUMBER` |  The ID of the container to which the data pertains. Possible values are: <br>* `0`: <br>This value is used for rows containing data that pertain to the entire multitenant container database (CDB). This value is also used for rows in non-CDBs. <br>`1`: <br>This value is used for rows containing data that pertain to only the root. <br>`n`: <br>Where `n` is the applicable container ID for the rows containing data.   
  
Example
```
    select CON_ID, POOL, ALLOC_BYTES/1024/1024 as ALLOC_BYTES_MB,
    USED_BYTES/1024/1024 as USED_BYTES_MB
    from V$VECTOR_MEMORY_POOL order by 1,2;
    
    
    CON_ID POOL             ALLOC_BYTES_MB USED_BYTES_MB
    ---------- ---------------- -------------- -------------
    1 1MB POOL                    319             0
    1 64KB POOL                   144             0
    1 IM POOL METADATA             32            32
    2 1MB POOL                    320             0
    2 64KB POOL                   144             0
    2 IM POOL METADATA             16            16
    3 1MB POOL                    320             0
    3 64KB POOL                   144             0
    3 IM POOL METADATA             16            16
    4 1MB POOL                    320             0
    4 64KB POOL                   144             0
    4 IM POOL METADATA             16            16
    
    12 rows selected.
    
    SQL>
```
    

**Parent topic:** [Vector Memory Pool Views](vector-memory-pool-views.md)
