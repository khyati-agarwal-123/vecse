## Vector Distance Functions and Operators {#GUID-8667D062-8084-4E55-8BD7-2C84E05F52FA}

A vector distance function takes in two vector operands and a distance metric to compute a mathematical distance between those two vectors, based on the distance metric provided. You can optionally use shorthand distance functions and operators instead of their corresponding distance functions.

Distances determine similarity or dissimilarity between vectors.

  * [Vector Distance Metrics](vector-distance-metrics.md)  
Measuring distances in a vector space is at the heart of identifying the most relevant results for a given query vector. That process is very different from the well-known keyword filtering in the relational database world. 
  * [VECTOR_DISTANCE](vector_distance.md)  
`VECTOR_DISTANCE` is the main function that you can use to calculate the distance between two vectors. 
  * [L1_DISTANCE](l1_distance.md)  
`L1_DISTANCE` is a shorthand version of the `VECTOR_DISTANCE` function that calculates the Manhattan distance between two vectors. It takes two vectors as input and returns the distance between them as a `BINARY_DOUBLE`. 
  * [L2_DISTANCE](l2_distance.md)  
`L2_DISTANCE` is a shorthand version of the `VECTOR_DISTANCE` function that calculates the Euclidean distance between two vectors. It takes two vectors as input and returns the distance between them as a `BINARY_DOUBLE`. 
  * [COSINE_DISTANCE](cosine_distance.md)  
`COSINE_DISTANCE` is a shorthand version of the `VECTOR_DISTANCE` function that calculates the cosine distance between two vectors. It takes two vectors as input and returns the distance between them as a `BINARY_DOUBLE`. 
  * [INNER_PRODUCT](inner_product.md)  
`INNER_PRODUCT` calculates the inner product of two vectors. It takes two vectors as input and returns the inner product as a `BINARY_DOUBLE`. `INNER_PRODUCT(<expr1>, <expr2>)` is equivalent to `-1 * VECTOR_DISTANCE(<expr1>, <expr2>, DOT)`. 
  * [HAMMING_DISTANCE](hamming_distance-vecse.md)  
`HAMMING_DISTANCE` is a shorthand version of the `VECTOR_DISTANCE` function that calculates the hamming distance between two vectors. It takes two vectors as input and returns the distance between them as a `BINARY_DOUBLE`. 
  * [JACCARD_DISTANCE](jaccard_distance-vecse.md)  
`JACCARD_DISTANCE` is a shorthand version of the `VECTOR_DISTANCE` function that calculates the jaccard distance between two vectors. It takes two `BINARY` vectors as input and returns the distance between them as a `BINARY_DOUBLE`. 



**Related Topics**

  * [Perform Exact Similarity Search](perform-exact-similarity-search.md#GUID-CCCF06F5-AD46-466D-99B2-4609B84C2B69)
  * [Perform Approximate Similarity Search Using Vector Indexes](perform-approximate-similarity-search-using-vector-indexes.md#GUID-D8432ADA-38B0-4E5F-975F-E86977CA8488)



**Parent topic:** [Use SQL Functions for Vector Operations](use-sql-functions-vector-operations.md)
