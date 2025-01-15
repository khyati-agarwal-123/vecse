##  VECTOR_NORM {#GUID-41554068-9EB8-49E8-A771-4E666674DDA8} 

` VECTOR_NORM ` returns the Euclidean norm of a vector ` (SQRT(SUM((xi-yi)2))) ` as a ` BINARY_DOUBLE ` . This value is also called magnitude or size and represents the Euclidean distance between the vector and the origin. 

Syntax 

  


![Description of vector_norm.eps follows](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img/vector_norm.gif)[ Description of the illustration vector_norm.eps ](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img_text/vector_norm.md)

  


Parameters 

*expr* must evaluate to a vector. 

If *expr* is NULL, NULL is returned. 

Example 
    
    
    ```
    
    SELECT VECTOR_NORM( TO_VECTOR('[4, 3]', 2, FLOAT32) );
    
    VECTOR_NORM(TO_VECTOR('[4,3]',2,FLOAT32))
    -----------------------------------------
    5.0E+000
    ```

**Parent topic:** [ Constructors, Converters, and Descriptors ](constructors-converters-descriptors-and-arithmetic-functions.md)
