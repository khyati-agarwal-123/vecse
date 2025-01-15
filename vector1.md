##  VECTOR {#GUID-8A63005B-5512-4D20-954C-7A9DA877FE4B} 

` VECTOR ` is synonymous with ` TO_VECTOR ` . 

Syntax 

  


![Description of vector_vecse.eps follows](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img/vector_vecse.gif)[ Description of the illustration vector_vecse.eps ](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img_text/vector_vecse.md)

  


Purpose 

See [ TO_VECTOR ](to_vector.html#GUID-2CCAB607-A28B-43F7-A71D-9800C0B9A380) for semantics and examples. 

> **note:** 

Applications using Oracle Client 23ai libraries or Thin mode drivers can insert vector data directly as a string or a ` CLOB ` . For example: 
    
    
    ```
    
    INSERT INTO vecTab VALUES ('[1.1, 2.9, 3.14]');
    ```

Examples 
    
    
    ```
    
    SELECT VECTOR('[34.6, 77.8]');
    
    VECTOR('[34.6,77.8]')
    ---------------------------------------------------------
    [3.45999985E+001,7.78000031E+001]
    
    
    
    SELECT VECTOR('[34.6, 77.8]', 2, FLOAT32);
    
    VECTOR('[34.6,77.8]',2,FLOAT32)
    ---------------------------------------------------------
    [3.45999985E+001,7.78000031E+001]
    
    
    
    SELECT VECTOR('[34.6, 77.8, -89.34]', 3, FLOAT32);
    
    VECTOR('[34.6,77.8,-89.34]',3,FLOAT32)
    -----------------------------------------------------------
    [3.45999985E+001,7.78000031E+001,-8.93399963E+001]
    ```

**Parent topic:** [ Vector Constructors ](vector-constructors.html)