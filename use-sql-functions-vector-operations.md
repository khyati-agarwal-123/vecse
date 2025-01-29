## 7Use SQL Functions for Vector Operations {#GUID-418342A8-1496-4668-B393-A3A8B91806A7}

There are a number of SQL functions and operators that you can use with vectors in Oracle AI Vector Search.

> **note:** You can also use PL/SQL packages to perform similar operations and additional tasks. See [Vector Search PL/SQL Packages](vector-search-pl-sql-packages-node.md#GUID-04ACF179-957C-4F03-AC1D-4DA44B3E12A2). 

  * [Vector Distance Functions and Operators](vector-distance-functions-and-operators.md)  
A vector distance function takes in two vector operands and a distance metric to compute a mathematical distance between those two vectors, based on the distance metric provided. You can optionally use shorthand distance functions and operators instead of their corresponding distance functions. 
  * [Chunking and Vector Generation Functions](chunking-and-vector-generation-functions.md)  
Oracle AI Vector Search offers Vector Utilities, which provide the `VECTOR_CHUNKS` and `VECTOR_EMBEDDING` SQL functions for chunking data and generating vector embedding, respectively. 
  * [Constructors, Converters, Descriptors, and Arithmetic Operators](constructors-converters-descriptors-and-arithmetic-operators.md)  
Other basic vector operations for Oracle AI Vector Search involve creating, converting, and describing vectors. 
  * [JSON Compatibility with the VECTOR Data Type](json-compatibility-vector-data-type.md)  
The JSON data type supports the `VECTOR` type as a JSON scalar type. A `VECTOR` instance is convertible to a JSON type instance and vice versa using the JSON constructor and the `VECTOR` constructor, respectively. 


