## Load Vector Data Using SQL*Loader {#GUID-FA5E79A3-B750-49C1-803E-9609CBDF7B91}

Use these examples to understand how you can load character and binary vector data.

SQL*Loader supports loading `VECTOR` columns from character data and binary floating point array `fvec` files. The format for `fvec` files is that each binary 32-bit floating point array is preceded by a four (4) byte value, which is the number of elements in the vector. There can be multiple vectors in the file, possibly with different dimensions. Export and import of a table with vector datatype columns is supported in all the modes ( FULL, SCHEMA, TABLES) using all the available methods (access_method=direct_path, access_method=external_table, access_method=automatic ) for unloading/loading data. 

  * [Load Character Vector Data Using SQL*Loader Example](load-character-vector-data-using-sqlloader-example.md)  
In this example, you can see how to use SQL*Loader to load vector data into a five-dimension vector space. 
  * [Load Binary Vector Data Using SQL*Loader Example](load-binary-vector-data-using-sqlloader-example.md)  
In this example, you can see how to use SQL*Loader to load binary vector data files. 



**Parent topic:** [Store Vector Embeddings](store-vector-embeddings.md)
