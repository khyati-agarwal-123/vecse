## Create Tables Using the VECTOR Data Type {#GUID-E05AC257-CBD6-4B0C-A29F-0116EF02EA3A}

You can declare a table's column as a `VECTOR` data type. 

The following command shows a simple example:
```
    CREATE TABLE my_vectors (id NUMBER, embedding VECTOR);
```
    

In this example, you don't have to specify the number of dimensions or their format, which are both optional. If you don't specify any of them, you can enter vectors of different dimensions with different formats. This is a simplification to help you get started with using vectors in Oracle Database.

> **note:** Such vectors typically arise from different embedding models. Vectors from different models (providing a different semantic landscape) are not comparable for use in similarity search. 

Here's a more complex example that imposes more constraints on what you can store:
```
    CREATE TABLE my_vectors (id NUMBER, embedding VECTOR(768, INT8)) ;
```
    

In this example, each vector that is stored:

  * Must have 768 dimensions, and
  * Each dimension will be formatted as an `INT8`. 



The number of dimensions must be strictly greater than zero with a maximum of 65535 for non-`BINARY` vectors and 65528 for `BINARY` vectors. If you attempt to use larger values, the following error is raised: 
```
    ORA-51801: Invalid VECTOR type specification: Invalid dimension
    count ('...'). Valid values can either be * (i.e. flexible) or an
    integer between 1 and 65535.
```
    

`BINARY` vectors must have a dimension that is a multiple of 8. If the specified dimension is not a multiple of 8, the following error is raised: 
```
    ORA-51813: Vector of BINARY format should have a dimension count
    that is a multiple of 8.
```
    

The possible dimension formats are:

  * `INT8` (8-bit integers) 
  * `FLOAT32` (32-bit IEEE floating-point numbers) 
  * `FLOAT64` (64-bit IEEE floating-point numbers) 
  * `BINARY` (packed `UINT8` bytes where each dimension is a single bit) 



Oracle Database automatically casts the values as needed.

Here is some simple code that shows you how to define and insert a `VECTOR` in a table: 
```
    DROP TABLE my_vect_tab PURGE;
    CREATE TABLE my_vect_tab (v01 VECTOR(3, INT8));
    INSERT INTO my_vect_tab VALUES ('[10, 20, 30]');
    
    SELECT * FROM my_vect_tab;
    
    V01
    ----------
    [10,20,30]
```
    

You use a textual form to represent a vector in SQL statements. The `DENSE` textual form as shown in the preceding `INSERT` statement is a basic example: coordinate 1 has a value of 10, coordinate 2 has a value of 20, and coordinate 3 has a value of 30. You separate each coordinate with a comma and you enclose the list with square brackets. 

So far, the vector types shown are, by default, `DENSE` vectors where each dimension is physically stored. All the definitions seen are equivalent to the following form: 
```
    VECTOR(number_of_dimensions, dimension_element_format, DENSE) or
    VECTOR(number_of_dimensions, dimension_element_format, *)
```
    

However, you also have the option to create `SPARSE` vectors. In contrast to `DENSE` vectors, a sparse vector is a vector whose dimension values are expected to be mostly zero. When using `SPARSE` vectors, only the non-zero values are physically stored. You define sparse vectors using the following form: 
```
    VECTOR(number_of_dimensions, dimension_element_format, SPARSE)
```
    

> **note:** 

  * It is not supported to have `SPARSE` storage in one row and `DENSE` storage in another row for the same vector column as the coding of the two representations are very different. 

  * Sparse vectors are not supported with `BINARY` format. 

  * `VECTOR(…, …, *)` is always interpreted as `DENSE`. 

  * Sparse vectors cannot currently be created or declared in PL/SQL.




The following table guides you through the possible declaration format for a `VECTOR` data type with a `DENSE` storage format: 

Possible Declaration Format  |  Explanation   
---|---  
`VECTOR` |  Vectors can have an arbitrary number of dimensions and formats.  
`VECTOR(*, *)`<br>`VECTOR(*, *, *)`<br>`VECTOR(*, *, DENSE)` |  Vectors can have an arbitrary number of dimensions and formats.<br>`VECTOR`, <br>`VECTOR(*, *)`, <br>`VECTOR(*,*,*)`, and <br>`VECTOR(*, *, DENSE)` are equivalent.   
`VECTOR`(number_of_dimensions) <br>`VECTOR`(number_of_dimensions, *) <br>`VECTOR`(number_of_dimensions, *, *) <br>`VECTOR`(number_of_dimensions, *, `DENSE`)  |  Vectors must all have the specified number of dimensions or an error is thrown. Every vector will have its dimensions stored without format modification.<br>`VECTOR`(number_of_dimensions), <br>`VECTOR`(number_of_dimensions, *), <br>`VECTOR`(number_of_dimensions, *, *), and <br>`VECTOR`(number_of_dimensions, *, `DENSE`) are equivalent.   
`VECTOR`(*, dimension_element_format) <br>`VECTOR`(*, dimension_element_format, *) <br>`VECTOR`(*, dimension_element_format, `DENSE`)  |  Vectors can have an arbitrary number of dimensions, but their format will be up-converted or down-converted to the specified dimension_element_format (`INT8`, `FLOAT32`, `FLOAT64`, or `BINARY`). <br>`VECTOR`(*, dimension_element_format), <br>`VECTOR`(*, dimension_element_format, *), and <br>`VECTOR`(*, dimension_element_format, `DENSE`) are equivalent.   
  
The following table guides you through the possible declaration format for a `VECTOR` data type for sparse vectors. 

Possible Declaration Format | Explanation  
---|---  
`VECTOR(*, *, SPARSE)` |  Vectors can have an arbitrary number of dimensions and formats besides `BINARY`.   
`VECTOR(number_of_dimensions, *, SPARSE)` |  Vectors must all have the specified number of dimensions or an error is thrown. Every vector will have its dimensions stored without format modification.  
`VECTOR(*, dimension_element_format, SPARSE)` |  Vectors can have an arbitrary number of dimensions but their format will be up-converted or down-converted to the specified `dimension_element_format` (`INT8`, `FLOAT32`, or `FLOAT64`)   
  
The following SQL*Plus code example shows how the system interprets various vector definitions:
```
    CREATE TABLE my_vect_tab (
    v1  VECTOR(3, FLOAT32),
    v2  VECTOR(2, FLOAT64),
    v3  VECTOR(1, INT8),
    v4  VECTOR(1024, BINARY),
    v5  VECTOR(1, *),
    v6  VECTOR(*, FLOAT32),
    v7  VECTOR(*, *),
    v8  VECTOR,
    v9  VECTOR(10),
    v10 VECTOR(*, *, DENSE),
    v11 VECTOR(1024, FLOAT32, DENSE),
    v12 VECTOR(1000, INT8, SPARSE),
    v13 VECTOR(*, INT8, SPARSE),
    v14 VECTOR(*, *, SPARSE),
    v15 VECTOR(2048, FLOAT32, *)
    );
    
    Table created.
    
    DESC my_vect_tab;
    Name                        Null?    Type
    --------------------------- -------- ----------------------------
    V1                                   VECTOR(3 , FLOAT32, DENSE)
    V2                                   VECTOR(2 , FLOAT64, DENSE)
    V3                                   VECTOR(1 , INT8, DENSE)
    V4                                   VECTOR(1024, BINARY, DENSE)
    V5                                   VECTOR(1 , *, DENSE)
    V6                                   VECTOR(* , FLOAT32, DENSE)
    V7                                   VECTOR(* , *, DENSE)
    V8                                   VECTOR(* , *, DENSE)
    v9                                   VECTOR(10, *, DENSE)
    v10                                  VECTOR(*, *, DENSE)
    v11                                  VECTOR(1024, FLOAT32, DENSE)
    v12                                  VECTOR(1000, INT8, SPARSE)
    v13                                  VECTOR(*, INT8, SPARSE)
    v14                                  VECTOR(*, *, SPARSE)
    v15                                  VECTOR(2048, FLOAT32, DENSE)
```
    

A vector can be `NULL` but its dimensions cannot (for example, you cannot have a `VECTOR` with a `NULL` dimension such as `[1.1, NULL, 2.2]`). 

> **note:** Vectors (`DENSE` and `SPARSE`) are internally stored as Securefile BLOBs and most popular embedding model vector sizes are between 1.5KB and 12KB in size. You can use the following formula to determine the size of your vectors on disk: 

  * For `DENSE` vectors: 

*number of vectors * number of dimensions * size of your vector dimension type* (for example, a `FLOAT32` is equivalent to `BINARY_FLOAT` and is 4 bytes in size). 

  * For `SPARSE` vectors: 

*number of vectors * ((average number of sparse dimensions * 4 bytes) + (number of sparse dimensions * size of your vector dimension type))*. 




Restrictions

You currently cannot define `VECTOR` columns in/as: 

  * IOTs (neither as Primary Key nor as non-Key column)
  * Clusters/Cluster Tables
  * Global Temp Tables
  * (Sub)Partitioning Key
  * Primary Key
  * Foreign Key
  * Unique Constraint
  * Check Constraint
  * Default Value
  * Modify Column
  * Manual Segment Space Management (MSSM) tablespace (only SYS user can create VECTORs as Basicfiles in MSSM tablespace)
  * Continuous Query Notification (CQN) queries
  * Non-vector indexes such as B-tree, Bitmap, Reverse Key, Text, Spatial indexes, etc



Oracle Database does not support the following SQL constructs with `VECTOR` columns: 

  * Distinct, Count Distinct
  * Order By, Group By
  * Join condition
  * Comparison operators (e.g. >, <, =) etc



Oracle's Globally Distributed Database has the following restrictions on vector data types:

  * Sharding keys: A distributed database only supports sharding keys on non-vector columns. The vector data can be distributed across shards using a primary key on any other non-vector column identified as a sharding key.

  * Raft replication: A distributed database using the Raft replication method for high availability does not support vector columns.




  * [Vectors in External Tables](vectors-external-tables.md)  
External tables can be created with columns of type `VECTOR`, allowing vector embeddings represented in text or binary format stored in external files to be rendered as the `VECTOR` data type in Oracle Database. 
  * [Vectors in Distributed Database Tables](vectors-distributed-database-tables.md)  
There is no new SQL syntax or keyword when creating sharded tables and duplicated tables with vector columns in a Globally Distributed Database; however, there are some requirements and restrictions to consider. 
  * [BINARY Vectors](binary-vectors.md)  
In addition to `FLOAT32` (the default format for comma-separated string representations of vectors), `FLOAT64`, and `INT8` dimension formats, you can also use the `BINARY` dimension format. 
  * [SPARSE Vectors](sparse-vectors.md)  
The storage format of a vector can be specified as `SPARSE` or `DENSE`. Sparse vectors are vectors that typically have a large number of dimensions but with very few non-zero dimension values, while Dense vectors are vectors where every dimension stores a value, including zero-values. 



**Parent topic:** [Store Vector Embeddings](store-vector-embeddings.md)
