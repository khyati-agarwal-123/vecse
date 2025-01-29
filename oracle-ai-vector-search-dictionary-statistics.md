## Oracle AI Vector Search Dictionary Statistics {#GUID-2E7E92AB-C1B6-4433-A4C7-064C6AA77FAA}

A set of dictionary statistics related to Oracle AI Vector Search.

  * **Vector simd dist single calls**: The number of vector distance function calls invoked by the user, where both the inputs are a single vector. 
  * **Vector simd dist point calls**: The number of vector distance function calls invoked by the user, where one input is a single vector, and the other input is an array of vectors. 
  * **Vector simd dist array calls**: The number of vector distance function calls invoked by the user, where both the inputs are an array of vectors. 
  * **Vector simd dist flex calls**: The number of vector distance function calls invoked by the user, where at least one input is with FLEX vector data type (no dimension or storage data type specified). 
  * **Vector simd dist total rows:** The number of rows processed by the vector distance function. 
  * **Vector simd dist flex rows**: The number of rows processed by the vector distance function with FLEX vector data type. 



  * **Vector simd topK single calls**: The number of `topK` distance function calls invoked by the user, where both the inputs are a single vector. 
  * **Vector simd topK point calls**: The number of `topK` distance function calls invoked by the user, where one input is a single vector and the other input is an array of vectors. 
  * **Vector simd topK array calls**: The number of `topK` distance function calls invoked by the user, where both the inputs are an array of vectors. 
  * **Vector simd topK flex calls**: The number of `topK` distance function calls invoked by the user, where at least one input is with FLEX vector data type (no dimension or storage data type specified). 
  * **Vector simd topK total rows**: The number of rows processed by the `topK` distance function. 
  * **Vector simd topK flex rows**: The number of rows processed by the `topK` distance function with FLEX vector data type. 
  * **Vector simd topK selected total rows**: The number of distance results returned by the `topK` vector distance function. 

> **note:** This is typically `sum(K)` for all `topK` queries. 




  * **Vector simd construction num of total calls**: The number of vector construction function calls invoked by the user. 
  * **Vector simd construction total result rows**: The number of vector rows returned by the vector construction function. 
  * **Vector simd construction num of ASCII calls**: The number of vector construction function calls invoked by the user, where the input encoding is ASCII. 
  * **Vector simd construction num of flex calls**: The number of vector construction function calls invoked by the user, where the output vector is of FLEX vector data type. 
  * **Vector simd construction result rows for flex**: The number of vector rows returned by the vector construction function, where the output vector is of FLEX vector data type. 



  * **Vector simd vector conversion num of total calls**: The number of vector conversion function calls invoked by the user. 



  * **VECTOR NEIGHBOR GRAPH HNSW build HT Element Allocated:** The number of hash table elements allocated for vector indexes having organization `Inmemory Neighbor Graph` and type `HNSW` that were created by a user. 
  * **VECTOR NEIGHBOR GRAPH HNSW build HT Element Freed:** The number of hash table elements freed for vector indexes having organization `Inmemory Neighbor Graph` and type `HNSW` that were created by a user. 
  * **VECTOR NEIGHBOR GRAPH HNSW reload attempted:** The number of reload operations attempted in the background for vector indexes having organization `Inmemory Neighbor Graph` and type `HNSW`. 
  * **VECTOR NEIGHBOR GRAPH HNSW reload successful:** The number of reload operations completed in the background for vector indexes having organization `Inmemory Neighbor Graph` and type `HNSW`. 
  * **VECTOR NEIGHBOR GRAPH HNSW build computed layers:** The total number of layers created in an HNSW index. 
  * **VECTOR NEIGHBOR GRAPH HNSW build indexed vectors:** The total number of vector indexes in the HNSW index. 
  * **VECTOR NEIGHBOR GRAPH HNSW build computed distances: ** The total number of distance computations executed during the HNSW index build phase. 
  * **VECTOR NEIGHBOR GRAPH HNSW build sparse layers computed distances:** The total number of distance computations executed during the HNSW index build phase, in all the layers, except the bottom one. 
  * **VECTOR NEIGHBOR GRAPH HNSW build dense layer computed distances:** The total number of distance computations executed during the HNSW index phase in the bottom layer. 
  * **VECTOR NEIGHBOR GRAPH HNSW build prune operation computed distances:** The total number of distance computations that were executed during the pruning operations required to find the closest neighbors for a vector in the HNSW index build phase. 
  * **VECTOR NEIGHBOR GRAPH HNSW build pruned neighbor lists:** The total number of neighbors pruned during the HNSW index build phase. 
  * **VECTOR NEIGHBOR GRAPH HNSW build pruned neighbors:** The total number of neighbors pruned during the HNSW index build phase. 
  * **VECTOR NEIGHBOR GRAPH HNSW build created edges:** The total number of edges created in an HNSW index. 
  * **VECTOR NEIGHBOR GRAPH HNSW search executed approximate:** The total number of query searches executed using approximate search in an HNSW index. 
  * **VECTOR NEIGHBOR GRAPH HNSW search executed exhaustive:** The total number of query searches executed using exact search in an HNSW index. 
  * **VECTOR NEIGHBOR GRAPH HNSW search computed distances dense layer:** The total number of distance computations executed in the bottom layer of an HNSW index during query searches. 
  * **VECTOR NEIGHBOR GRAPH HNSW search computed distances sparse layer:** The total number of distance computations executed in all the layers except the bottom layer of an HNSW index during query searches. 



  * **VECTOR NEIGHBOR PARTITIONS IVF build HT Element Allocated:** The number of hash table elements allocated for vector indexes having organization `Neighbor Partitions` and type `IVF` that were created by a user. 
  * **VECTOR NEIGHBOR PARTITIONS IVF build HT Element Freed:** The number of hash table elements freed for vector indexes having organization `Neighbor Partitions` and type `IVF` that were created by a user. 
  * **VECTOR NEIGHBOR PARTITIONS IVF background Population Started:** The number of in-memory centroids background population operations started for vector indexes having organization `Neighbor Partitions` and type `IVF`. 
  * **VECTOR NEIGHBOR PARTITIONS IVF background Population Succeeded:** The number of in-memory centroids background population operations completed for vector indexes having organization `Neighbor Partitions` and type `IVF`. 
  * **VECTOR NEIGHBOR PARTITIONS IVF background Cleanup Started:** The number of in-memory centroids background cleanup operations started for vector indexes having organization `Neighbor Partitions` and type `IVF`. 
  * **VECTOR NEIGHBOR PARTITIONS IVF XIC Population Started:** The number of in-memory centroids cross-instance population operations started for vector indexes having organization `Neighbor Partitions` and type `IVF`. 
  * **VECTOR NEIGHBOR PARTITIONS IVF XIC Population Succeeded:** The number of in-memory centroids cross-instance population operations completed for vector indexes having organization `Neighbor Partitions` and type `IVF`. 
  * **VECTOR NEIGHBOR PARTITIONS IVF XIC Cleanup Started:** The number of in-memory centroids cross-instance cleanup operations started for vector indexes having organization `Neighbor Partitions` and type `IVF`. 
  * **VECTOR NEIGHBOR PARTITIONS IVF XIC Cleanup Succeeded:** The number of in-memory centroids cross-instance cleanup operations completed for vector indexes having organization `Neighbor Partitions` and type `IVF`. 
  * **VECTOR NEIGHBOR PARTITIONS IVF im centroids in scan:** The number of times that in-memory centroids are used in scan for vector indexes having organization `Neighbor Partitions` and type `IVF`. 
  * **VECTOR NEIGHBOR PARTITIONS IVF im centroids in dmls:** The number of times that in-memory centroids are used in the DML that happens on the base table with vector indexes having organization `Neighbor Partitions` and type `IVF`. 



**Parent topic:** [Oracle AI Vector Search Statistics](oracle-ai-vector-search-statistics.md)
