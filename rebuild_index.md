## REBUILD_INDEX {#GUID-7A7B63F6-97E3-44C6-8CD4-C84376F06F14}

Use the `DBMS_VECTOR.REBUILD_INDEX` function to rebuild a vector index. 

Purpose

To rebuild a vector index such as Hierarchical Navigable Small World (HNSW) vector index or Inverted File Flat (IVF) vector index. In case only the `idx_name` is provided, it rebuilds the index using `get_ddl`. When all the parameters are provided, it performs a drop index followed by a call to `dbms_vector.create_index()`. 

Syntax
```
    DBMS_VECTOR.REBUILD_INDEX (
    idx_name                   IN VARCHAR2,
    table_name                 IN VARCHAR2 DEFAULT NULL,
    idx_vector_col             IN VARCHAR2 DEFAULT NULL,
    idx_include_cols           IN VARCHAR2 DEFAULT NULL,
    idx_partitioning_scheme    IN VARCHAR2 DEFAULT NULL,
    idx_organization           IN VARCHAR2 DEFAULT NULL,
    idx_distance_metric        IN VARCHAR2 DEFAULT 'COSINE',
    idx_accuracy               IN NUMBER DEFAULT 90,
    idx_parameters             IN CLOB DEFAULT NULL,
    idx_parallel_creation      IN NUMBER DEFAULT 1,
    );
```
    

Parameters

Parameter | Description  
---|---  
`idx_name` |  Name of the index to rebuild.  
`table_name` |  Table on which to create the index.  
`idx_vector_col ` |  Vector column on which to create the index.  
`idx_include_cols` |  A comma-separated list of column names to be covered by the index.  
`idx_partitioning_scheme` |  Partitioning scheme for IVF indexes: <br>* `GLOBAL` <br>`LOCAL`
<br>IVF indexes support both global and local indexes on partitioned tables. By default, these indexes are globally partitioned by centroid. You can choose to create a local IVF index, which provides a one-to-one relationship between the base table partitions or subpartitions and the index partitions.<br>For detailed information on these partitioning schemes, see [Inverted File Flat Vector Indexes Partitioning Schemes](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-98797C70-269E-42E9-B24B-1E3C461D5932).   
`idx_organization` |  Index organization: <br>* `NEIGHBOR PARTITIONS` <br>`INMEMORY NEIGHBOR GRAPH`
<br>For detailed information on these organization types, see [Manage the Different Categories of Vector Indexes](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-5D9B6B92-C62C-4927-9FB2-7A4437F24A19).   
`idx_distance_metric` |  Distance metric or mathematical function used to compute the distance between vectors: <br>* `COSINE` (default) <br>`MANHATTAN`<br>`HAMMING`<br>`JACCARD`<br>`DOT`<br>`EUCLIDEAN`<br>`L2_SQUARED`<br>`EUCLIDEAN_SQUARED`
<br>For detailed information on each of these metrics, see [Vector Distance Functions and Operators](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-8667D062-8084-4E55-8BD7-2C84E05F52FA).   
`idx_accuracy` |  Target accuracy at which the approximate search should be performed when running an approximate search query.<br>As explained in [Understand Approximate Similarity Search Using Vector Indexes](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-69B32E29-4ACA-4483-8607-BEDC2EFF6EEB), you can specify non-default target accuracy values either by specifying a percentage value or by specifying internal parameters values, depending on the index type you are using. <br>* For an HNSW approximate search: <br>In the case of an HNSW approximate search, you can specify a target accuracy percentage value to influence the number of candidates considered to probe the search. This is automatically calculated by the algorithm. A value of 100 will tend to impose a similar result as an exact search, although the system may still use the index and will not perform an exact search. The optimizer may choose to still use an index as it may be faster to do so given the predicates in the query. Instead of specifying a target accuracy percentage value, you can specify the `EFSEARCH` parameter to impose a certain maximum number of candidates to be considered while probing the index. The higher that number, the higher the accuracy. <br>For detailed information, see [Understand Hierarchical Navigable Small World Indexes](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-23E74FFC-B6A5-4374-9813-F2634AC43410).  <br>For an IVF approximate search: <br>In the case of an IVF approximate search, you can specify a target accuracy percentage value to influence the number of partitions used to probe the search. This is automatically calculated by the algorithm. A value of 100 will tend to impose an exact search, although the system may still use the index and will not perform an exact search. The optimizer may choose to still use an index as it may be faster to do so given the predicates in the query. Instead of specifying a target accuracy percentage value, you can specify the `NEIGHBOR PARTITION PROBES` parameter to impose a certain maximum number of partitions to be probed by the search. The higher that number, the higher the accuracy. <br>For detailed information, see [Understand Inverted File Flat Vector Indexes](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-332E5EA7-6CE9-4392-AB2B-3124BF832C5E).   
  
`idx_parameters` |  Type of vector index and associated parameters.<br>Specify the indexing parameters in JSON format: <br>* For HNSW indexes: <br>* `type`: <br>Type of vector index to create, that is, `HNSW` <br>`neighbors`: <br>Maximum number of connections permitted per vector in the HNSW graph  <br>`efConstruction`: <br>Maximum number of closest vector candidates considered at each step of the search during insertion 
<br>For example: <br>```{ "type" : <br>"HNSW", "neighbors" : <br>3, "efConstruction" : <br>4 }```<br>For detailed information on these parameters, see [Hierarchical Navigable Small World Index Syntax and Parameters](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-0D538811-D59F-46FB-9453-1A6BD822EEED).  <br>For IVF indexes: <br>* `type`: <br>Type of vector index to create, that is, `IVF` <br>`partitions`: <br>Neighbor partition or cluster in which you want to divide your vector space 
<br>For example: <br>```{ "type" : <br>"IVF", "partitions" : <br>5 }```<br>For detailed information on these parameters, see [Inverted File Flat Index Syntax and Parameters](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-FC314C40-1018-46B9-9F1C-660BBE28FBE9).   
  
`idx_parallel_creation` |  Number of parallel threads used for index construction.  
  
Examples

  * Specify neighbors and efConstruction for HNSW indexes:
```
    dbms_vector.rebuild_index(
    'v_hnsw_01',
    'vpt01',
    'EMBEDDING',
    NULL,
    NULL,
    'INMEMORY NEIGHBOR GRAPH',
    'EUCLIDEAN',
    95,
    '{"type" : "HNSW", "neighbors" : 3, "efConstruction" : 4}');
```
    

  * Specify the number of partitions for IVF indexes:
```
    dbms_vector.rebuild_index(
    'V_IVF_01',
    'vpt01',
    'EMBEDDING',
    NULL,
    NULL,
    'NEIGHBOR PARTITIONS',
    'EUCLIDEAN',
    95,
    '{"type" : "IVF", "partitions" : 5}');
```
    




**Parent topic:** [DBMS_VECTOR](dbms_vector-vecse.md)
