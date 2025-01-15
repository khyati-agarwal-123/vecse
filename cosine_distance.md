##  COSINE_DISTANCE {#GUID-2128DC1D-612A-444F-87D8-3D249CD8F12D} 

` COSINE_DISTANCE ` is a shorthand version of the ` VECTOR_DISTANCE ` function that calculates the cosine distance between two vectors. It takes two vectors as input and returns the distance between them as a ` BINARY_DOUBLE ` . 

Syntax 

  


![Description of cosine_distance.eps follows](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img/cosine_distance.gif)[ Description of the illustration cosine_distance.eps ](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img_text/cosine_distance.md)

  


Parameters 

  * *expr1* and *expr2* must evaluate to vectors that have the same format and number of dimensions. 

  * ` COSINE_DISTANCE ` returns NULL, if either *expr1* or *expr2* is NULL. 




**Parent topic:** [ Vector Distance Functions and Operators ](vector-distance-functions-and-operators.md)
