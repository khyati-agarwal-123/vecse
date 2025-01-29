## Optimizer Plans for Vector Indexes {#GUID-80BBB84C-48F4-4C0B-9137-0B114D5515C9}

Optimizer plans for HNSW and IVF indexes are described in the following sections.

  * [Optimizer Plans for HNSW Vector Indexes](optimizer-plans-hnsw-vector-indexes.md)  
A Hierarchical Navigable Small World Graph (HNSW) is a form of In-Memory Neighbor Graph vector index. It is a very efficient index for vector approximate similarity search. 
  * [Optimizer Plans for IVF Vector Indexes](optimizer-plans-ivf-vector-indexes.md)  
Inverted File Flat (IVF) is a form of Neighbor Partition Vector index. It is a partition-based index that achieves search efficiency by narrowing the search area through the use of neighbor partitions or clusters. 
  * [Vector Index Hints](vector-index-hints.md)  
If the optimizer does not choose your existing index while running your query and you still want that index to be used, you can rewrite your SQL statement to include vector index hints. 



**Parent topic:** [Perform Approximate Similarity Search Using Vector Indexes](perform-approximate-similarity-search-using-vector-indexes.md)
