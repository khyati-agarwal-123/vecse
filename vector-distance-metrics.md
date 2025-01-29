## Vector Distance Metrics {#GUID-DBC136C1-7C63-4B7F-902B-2289FF375560}

Measuring distances in a vector space is at the heart of identifying the most relevant results for a given query vector. That process is very different from the well-known keyword filtering in the relational database world.

When working with vectors, there are several ways you can calculate distances to determine how similar, or dissimilar, two vectors are. Each distance metric is computed using different mathematical formulas. The time it takes to calculate the distance between two vectors depends on many factors, including the distance metric used as well as the format of the vectors themselves, such as the number of vector dimensions and the vector dimension formats. Generally it's best to match the distance metric you use to the one that was used to train the vector embedding model that generated the vectors.

  * [Euclidean and Euclidean Squared Distances](euclidean-and-squared-euclidean-distances.md)  
Euclidean distance reflects the distance between each of the vectors' coordinates being compared—basically the straight-line distance between two vectors. This is calculated using the Pythagorean theorem applied to the vector's coordinates `(SQRT(SUM((xi-yi)2)))`. 
  * [Cosine Similarity](cosine-similarity.md)  
One of the most widely used similarity metric, especially in natural language processing (NLP), is cosine similarity, which measures the cosine of the angle between two vectors. 
  * [Dot Product Similarity](dot-product-similarity.md)  
The dot product similarity of two vectors can be viewed as multiplying the size of each vector by the cosine of their angle. The corresponding geometrical interpretation of this definition is equivalent to multiplying the size of one of the vectors by the size of the projection of the second vector onto the first one, or vice versa. 
  * [Manhattan Distance](manhattan-distance.md)  
This metric is calculated by summing the distance between the dimensions of the two vectors that you want to compare. 
  * [Hamming Distance](hamming-distance.md)  
The Hamming distance between two vectors represents the number of dimensions where they differ. 
  * [Jaccard Similarity](jaccard-similarity.md)  
The Jaccard similarity is used to determine the share of significant (non-zero) dimensions (bit's position) common between two `BINARY` vectors. 



**Parent topic:** [Vector Distance Functions and Operators](vector-distance-functions-and-operators.md)
