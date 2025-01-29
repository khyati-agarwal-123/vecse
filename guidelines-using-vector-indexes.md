## Guidelines for Using Vector Indexes {#GUID-F77D534C-2467-47A7-A5A0-70AE0957213A}

Use these guidelines to create and use Hierarchical Navigable Small World (HNSW) or Inverted File Flat (IVF) vector indexes.

Create Index Guidelines

The minimum information required to create a vector index is to specify one `VECTOR` data type table column and a vector index type: `INMEMORY NEIGHBOR GRAPH` for HNSW and `NEIGHBOR PARTITIONS` for IVF. However, you also have the possibility to specify more information, such as the following: 

  * You can optionally provide more information including the distance metric to use. Supported metrics are `EUCLIDEAN SQUARED`, `EUCLIDEAN`, `COSINE`, `DOT`, `MANHATTAN`, and `HAMMING`. If not specified, `COSINE` is used by default. 
  * Specific parameters that impact the accuracy of index creation and approximate searches. A target accuracy percentage value and, `NEIGHBORS` (or M) and `EFCONSTRUCTION` for HNSW and `NEIGHBOR PARTITIONS` for IVF. 
  * You can create globally partitioned vector indexes.
  * You can also specify the degree of parallelism to use for index creation.



You cannot currently define a vector index on:

  * External tables
  * IOTs
  * Clusters/Cluster tables
  * Global Temp tables
  * Blockchain tables
  * Immutable tables
  * Materialized views
  * Function-based vector index



You can find information about your vector indexes by looking at `ALL_INDEXES`, `DBA_INDEXES`, and `USER_INDEXES` family of views. The columns of interest are `INDEX_TYPE` (`VECTOR`) and `INDEX_SUBTYPE` (`INMEMORY_NEIGHBOR_GRAPH_HNSW` or `NEIGHBOR_PARTITIONS_IVF`). In the case the index is not a vector index, `INDEX_SUBTYPE` is `NULL`. 

See [VECSYS.VECTOR$INDEX](vecsys-vectorindex.md#GUID-FA5183EA-3C60-470D-9C91-8215BE4FF138) for detailed information about vector indexes. 

> **note:** 

  * The `VECTOR` column is designed to be extremely flexible to support vectors of any number of dimensions and any format for the vector dimensions. However, you can create a vector index only on a `VECTOR` column containing vectors that all have the same number of dimensions. This is required as you can't compute distances over vectors with different dimensions. For example, if a `VECTOR` column is defined as `VECTOR(*, FLOAT32)`, and two vectors with different dimensions (128 and 256 respectively) are inserted in that column. When you try to create the vector index on that column, you will get an error. 
  * Once an IVF index is created on a vector column of an empty table and non-null vectors with a particular dimension have been inserted into it, new vectors with a different dimension cannot be inserted into the same vector column due to the existence of the IVF index.
  * If a table is truncated, any IVF index created on the table is marked as `UNUSABLE`. Thus, the IVF index will not be maintained on any subsequent `INSERT` statements with vectors of the same or different dimension count. You must rebuild or recreate the IVF index before using the index in a query again. 
  * You can only create one type of vector index per vector column.
  * Oracle recommends that you allocate larger, temporary tablespaces for proper IVF vector index creation with big vector spaces and vector sizes. In such cases, the system internally makes extensive use of temporary space.
  * On a RAC environment you can set up a vector pool on each instance for the best performance of vector indexes.



Use Index Guidelines

For the Oracle Database Optimizer to consider a vector index, you must ensure to verify these conditions in your SQL statements:

  * The similarity search SQL query must include the `APPROX` or `APPROXIMATE` keyword. 
  * The vector index must exist.
  * The distance function for the index must be the same as the distance function used in the `vector_distance()` function. 
  * If the vector index DDL does not specify the distance function and the `vector_distance()` function uses `COSINE`, `DOT`, `MANHATTAN` or `HAMMING`, then the vector index is not used. 
  * If the vector index DDL uses the `DOT` distance function and the `vector_distance()` function uses the default distance function `[COSINE]`, then the vector index is not used. 
  * The `vector_distance()` must not be encased in another SQL function. 
  * If using the partition row-limiting clause, then the vector index is not used.
  * Index accuracy with an IVF index may diminish over time due to DML operations being performed on the underlying table. You can check for this by using the `INDEX_ACCURACY_QUERY` function provided by the `DBMS_VECTOR` package. In such a case, the index can be rebuild using the `REBUILD_INDEX` function also provided by the `DBMS_VECTOR` package. See [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12) for more information about `DBMS_VECTOR` and its subprograms. 



> **note:** Except for `ALTER TABLE tab SUBPARTITION`/`PARTITION` (`RANGE`/`LIST`) with or without Update Global Indexes, global vector indexes are marked as unusable as part of other Partition Management Operations on a partitioned table. In those cases you must manually rebuild the global vector indexes. 

**Parent topic:** [Manage the Different Categories of Vector Indexes](manage-different-categories-vector-indexes.md)
