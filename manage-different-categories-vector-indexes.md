## Manage the Different Categories of Vector Indexes {#GUID-5D9B6B92-C62C-4927-9FB2-7A4437F24A19}

There are two ways to make vector searches faster:

  * Reduce the search scope by clustering vectors (nearest neighbors) into structures based on certain attributes and restricting the search to closest clusters.
  * Reduce the vector size by reducing the number of bits representing vectors values.



Oracle AI Vector Search supports the following categories of vector indexing methods based on approximate nearest-neighbors (ANN) search:

  * In-Memory Neighbor Graph Vector Index
  * Neighbor Partition Vector Index



The distance function used to create and search the index should be the one recommended by the embedding model used to create the vectors. You can specify this distance function at the time of index creation or when you perform a similarity search using the `VECTOR_DISTANCE()` function. If you use a different distance function than the one used to create the index, an exact match is triggered because you cannot use the index in this case. 

> **note:** 

  * Oracle AI Vector Search indexes supports the same distance metrics as the `VECTOR_DISTANCE()` function. `COSINE` is the default metric if you do not specify any metric at the time of index creation or during a similarity search using the `VECTOR_DISTANCE()` function. 
  * You should always define the distance metric in an index based on the distance metric used by the embedding model you are using.



  * [In-Memory Neighbor Graph Vector Index](memory-neighbor-graph-vector-index.md)  
Hierarchical Navigable Small World (HNSW) is the only type of In-Memory Neighbor Graph vector index supported. HNSW graphs are very efficient indexes for vector approximate similarity search. HNSW graphs are structured using principles from small world networks along with layered hierarchical organization. 
  * [Neighbor Partition Vector Index](neighbor-partition-vector-index.md)  
Inverted File Flat (IVF) index is the only type of Neighbor Partition vector index supported. Inverted File Flat Index (IVF Flat or simply IVF) is a partitioned-based index that lets you balance high-search quality with reasonable speed. 
  * [Guidelines for Using Vector Indexes](guidelines-using-vector-indexes.md)  
Use these guidelines to create and use Hierarchical Navigable Small World (HNSW) or Inverted File Flat (IVF) vector indexes. 
  * [Index Accuracy Report](index-accuracy-report.md)  
The index accuracy reporting feature lets you determine the accuracy of your vector indexes. 
  * [Vector Index Status, Checkpoint, and Advisor Procedures](vector-index-status-checkpoint-and-advisor-procedures.md)  
Review these high-level details on the `GET_INDEX_STATUS`, `ENABLE_CHECKPOINT`, `DISABLE_CHECKPOINT`, and `INDEX_VECTOR_MEMORY_ADVISOR` procedures that are available with the `DBMS_VECTOR` PL/SQL package. 



**Parent topic:** [Create Vector Indexes and Hybrid Vector Indexes](create-vector-indexes-and-hybrid-vector-indexes.md)
