## INNER_PRODUCT {#GUID-6AE745CF-93E7-4192-8F80-7B9853DF5B72}

`INNER_PRODUCT` calculates the inner product of two vectors. It takes two vectors as input and returns the inner product as a `BINARY_DOUBLE`. `INNER_PRODUCT(<expr1>, <expr2>)` is equivalent to `-1 * VECTOR_DISTANCE(<expr1>, <expr2>, DOT)`. 

Syntax

  


![Description of inner_product.eps follows](img/inner_product.gif)  


  


Parameters

  * *expr1* and *expr2* must evaluate to vectors that have the same format and number of dimensions. 

  * `INNER_PRODUCT` returns NULL, if either *expr1* or *expr2* is NULL. 




**Parent topic:** [Vector Distance Functions and Operators](vector-distance-functions-and-operators.md)
