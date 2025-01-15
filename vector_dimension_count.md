##  VECTOR_DIMENSION_COUNT {#GUID-C3D937E0-7F9F-4C21-A214-0CFA31472E67} 

` VECTOR_DIMENSION_COUNT ` returns the number of dimensions of a vector as a ` NUMBER ` . 

Syntax 

  


![Description of vector_dimension_count.eps follows](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img/vector_dimension_count.gif)[ Description of the illustration vector_dimension_count.eps ](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img_text/vector_dimension_count.md)

  


Purpose 

` VECTOR_DIMENSION_COUNT ` is synonymous with [ VECTOR_DIMS ](vector_dims.md#GUID-010349D7-190D-430B-A798-ACC486E1036A) . 

Parameters 

*expr* must evaluate to a vector. 

If *expr* is NULL, NULL is returned. 

Example 
    
    
    ```
    
    SELECT VECTOR_DIMENSION_COUNT( TO_VECTOR('[34.6, 77.8]', 2, FLOAT64) );
    
    VECTOR_DIMENSION_COUNT(TO_VECTOR('[34.6,77.8]',2,FLOAT64))
    ----------------------------------------------------------
    2                          
    
    ```

**Parent topic:** [ Constructors, Converters, and Descriptors ](constructors-converters-descriptors-and-arithmetic-functions.md)
