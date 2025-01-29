## Hierarchical Navigable Small World Index Syntax and Parameters {#GUID-0D538811-D59F-46FB-9453-1A6BD822EEED}

Review the syntax and examples for Hierarchical Navigable Small World vector indexes.

Syntax
```
    CREATE VECTOR INDEX vector_index_name
    ON table_name (vector_column)
    [GLOBAL] ORGANIZATION INMEMORY [NEIGHBOR] GRAPH
    [WITH] [DISTANCE metric name]
    [WITH TARGET ACCURACY percentage_value]
    [PARAMETERS (TYPE HNSW , {NEIGHBORS max_closest_vectors_connected
    \| M max_closest_vectors_connected}
    , EFCONSTRUCTION max_candidates_to_consider)]
    [PARALLEL degree_of_parallelism]
```
    

HNSW Parameters

NEIGHBORS and M are equivalent and represent the maximum number of neighbors a vector can have on any layer. The last vertex has one additional flexibility that it can have up to 2M neighbors. 

EFCONSTRUCTION represents the maximum number of closest vector candidates considered at each step of the search during insertion. 

The valid range for HNSW vector index parameters are:

  * `ACCURACY`: > 0 and <= 100 
  * `DISTANCE`: `EUCLIDEAN`, `L2_SQUARED` (aka `EUCLIDEAN_SQUARED`), `COSINE`, `DOT`, `MANHATTAN`, `HAMMING`
  * `TYPE`: `HNSW`
  * `NEIGHBORS`: > 0 and <= 2048 
  * `EFCONSTRUCTION`: > 0 and <= 65535 



Examples
```
    CREATE VECTOR INDEX galaxies_hnsw_idx ON galaxies (embedding) ORGANIZATION INMEMORY NEIGHBOR GRAPH
    DISTANCE COSINE
    WITH TARGET ACCURACY 95;
    
    CREATE VECTOR INDEX galaxies_hnsw_idx ON galaxies (embedding) ORGANIZATION INMEMORY NEIGHBOR GRAPH
    DISTANCE COSINE
    WITH TARGET ACCURACY 90 PARAMETERS (type HNSW, neighbors 40, efconstruction 500);
```
    

For detailed information, see [CREATE VECTOR INDEX](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SQLRF-GUID-B396C369-54BB-4098-A0DD-7C54B3A0D66F) in *Oracle Database SQL Language Reference*. 

**Parent topic:** [In-Memory Neighbor Graph Vector Index](memory-neighbor-graph-vector-index.md)
