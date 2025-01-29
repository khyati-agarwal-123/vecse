## L1_DISTANCE {#GUID-604A5B68-10AF-48F3-A84F-ED0B90624059}

`L1_DISTANCE` is a shorthand version of the `VECTOR_DISTANCE` function that calculates the Manhattan distance between two vectors. It takes two vectors as input and returns the distance between them as a `BINARY_DOUBLE`. 

Syntax

  


![Description of l1_distance.eps follows](img/l1_distance.gif)  


  


Parameters

  * *expr1* and *expr2* must evaluate to vectors and have the same format and number of dimensions. 

  * `L1_DISTANCE` returns NULL, if either *expr1* or *expr2* is NULL. 




**Parent topic:** [Vector Distance Functions and Operators](vector-distance-functions-and-operators.md)
