## VECTOR_DIMENSION_FORMAT {#GUID-354ACE80-7120-4D45-B2B0-AB1D86E3D37D}

`VECTOR_DIMENSION_FORMAT` returns the storage format of the vector. It returns a `VARCHAR2`, which can be one of the following values: `INT8`, `FLOAT32`, `FLOAT64`, or `BINARY`. 

Syntax

<br>![Description of vector_dimension_format.eps follows](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img/vector_dimension_format.gif)<br>[Descriptionof the illustration vector_dimension_format.eps](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img_text/vector_dimension_format.md)

<br>Parameters

*expr* must evaluate to a vector. 

If *expr* is NULL, NULL is returned. 

Examples
```
SELECT VECTOR_DIMENSION_FORMAT(TO_VECTOR('[34.6, 77.8]', 2, FLOAT64));

VECTOR_DIMENSION_FORMAT(TO_VECTOR('[34.6,77.8]',2,
--------------------------------------------------
FLOAT64


SELECT VECTOR_DIMENSION_FORMAT(TO_VECTOR('[34.6, 77.8, 9]', 3, FLOAT32));

VECTOR_DIMENSION_FORMAT(TO_VECTOR('[34.6,77.8,9]',
--------------------------------------------------
FLOAT32


SELECT VECTOR_DIMENSION_FORMAT(TO_VECTOR('[34.6, 77.8, 9.10]', 3, INT8));

VECTOR_DIMENSION_FORMAT(TO_VECTOR('[34.6,77.8,9.10
--------------------------------------------------
INT8


SELECT VECTOR_DIMENSION_FORMAT(TO_VECTOR('[206, 32]', 16, BINARY));

VECTOR_DIMENSION_FORMAT(TO_VECTOR('[206,32]',16,BI
--------------------------------------------------
BINARY


SELECT VECTOR_DIMENSION_FORMAT(TO_VECTOR('[34.6, 77.8, 9, 10]', 3, INT8));

SELECT VECTOR_DIMENSION_FORMAT(TO_VECTOR('[34.6, 77.8, 9, 10]', 3, INT8))
*
ERROR at line 1:
ORA-51803: Vector dimension count must match the dimension count specified in
the column definition (expected 3 dimensions, specified 4 dimensions).
```


**Parent topic:** [Constructors, Converters, and Descriptors](constructors-converters-descriptors-and-arithmetic-functions.md)