## Vector Constructors {#GUID-2B9F10B9-01E7-4C9B-A8B6-E1B7D51F737D}

`TO_VECTOR()` and `VECTOR()` are synonymous constructors of vectors. The functions take a string of type `VARCHAR2` or `CLOB` as input and return a vector as output. 

* [TO_VECTOR](to_vector.md)<br>`TO_VECTOR` is a constructor that takes a string of type `VARCHAR2`, `CLOB`, `BLOB`, or `JSON` as input, converts it to a vector, and returns a vector as output. `TO_VECTOR` also takes another vector as input, adjusts its format, and returns the adjusted vector as output. `TO_VECTOR` is synonymous with `VECTOR`. 
* [VECTOR](vector1.md)<br>`VECTOR` is synonymous with `TO_VECTOR`. 



**Parent topic:** [Constructors, Converters, and Descriptors](constructors-converters-descriptors-and-arithmetic-functions.md)