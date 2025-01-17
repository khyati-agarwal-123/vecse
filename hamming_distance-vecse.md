## HAMMING_DISTANCE {#GUID-0028B83A-82C0-4B53-A783-23EFD90DF147}

`HAMMING_DISTANCE` is a shorthand version of the `VECTOR_DISTANCE` function that calculates the hamming distance between two vectors. It takes two vectors as input and returns the distance between them as a `BINARY_DOUBLE`. 

Syntax

<br>![Description of hamming_distance_syntax.eps follows](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img/hamming_distance_syntax.gif)<br>[Descriptionof the illustration hamming_distance_syntax.eps](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img_text/hamming_distance_syntax.md)

<br>Parameters

* *expr1* and *expr2* must evaluate to vectors that have the same format and number of dimensions. 

* `HAMMING_DISTANCE` returns NULL if either *expr1* or *expr2* is NULL. 




**Parent topic:** [Vector Distance Functions and Operators](vector-distance-functions-and-operators.md)