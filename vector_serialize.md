##  VECTOR_SERIALIZE {#GUID-9E3FFB34-F924-4C02-B35D-30B9FA1DA1A3} 

` VECTOR_SERIALIZE ` is synonymous with ` FROM_VECTOR. `

Syntax 

  


![Description of vector_serialize.eps follows](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img/vector_serialize.gif)[ Description of the illustration vector_serialize.eps ](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img_text/vector_serialize.md)

  


Purpose 

See [ FROM_VECTOR ](from_vector.md#GUID-AA60B3CB-FCB7-4944-9E06-976C272855B1) for semantics and examples. 

Examples 
    
    
    ```
    
    SELECT VECTOR_SERIALIZE(VECTOR('[1.1,2.2,3.3]',3,FLOAT32));
    
    
    VECTOR_SERIALIZE(VECTOR('[1.1,2.2,3.3]',3,FLOAT32))
    ---------------------------------------------------------------
    [1.10000002E+000,2.20000005E+000,3.29999995E+000]
    
    1 row selected.
    
    ```
    
    
    ```
    
    SELECT VECTOR_SERIALIZE(VECTOR('[1.1, 2.2, 3.3]',3,FLOAT32) RETURNING VARCHAR2(1000));
    
    
    VECTOR_SERIALIZE(VECTOR('[...]',3,FLOAT32)RETURNINGVARCHAR2(1000))
    ------------------------------------------------------------------
    [1.10000002E+000,2.20000005E+000,3.29999995E+000]
    
    1 row selected.
    ```
    
    
    ```
    
    SELECT VECTOR_SERIALIZE(VECTOR('[1.1, 2.2, 3.3]',3,FLOAT32) RETURNING CLOB);
    
    
    VECTOR_SERIALIZE(VECTOR('[1.1, 2.2, 3.3]',3,FLOAT32)RETURNINGCLOB)
    --------------------------------------------------------
    [1.10000002E+000,2.20000005E+000,3.29999995E+000] 
    
    1 row selected.
    
    ```
    
    
    ```
    
    SELECT VECTOR_SERIALIZE(TO_VECTOR('[5,[2,4],[1.0,2.0]]', 5, FLOAT64, SPARSE) RETURNING CLOB FORMAT SPARSE);
    
    VECTOR_SERIALIZE(TO_VECTOR('[5,[2,4],[1.0,2.0]]',5,FLOAT64,SPARSE)RETURNINGCLOBF
    --------------------------------------------------------------------------------
    [5,[2,4],[1.0E+000,2.0E+000]]
    
    1 row selected.
    ```
    
    
    ```
    
    SELECT VECTOR_SERIALIZE(TO_VECTOR('[5,[2,4],[1.0,2.0]]', 5, FLOAT64, SPARSE) RETURNING CLOB FORMAT DENSE);
    
    VECTOR_SERIALIZE(TO_VECTOR('[5,[2,4],[1.0,2.0]]',5,FLOAT64,SPARSE)RETURNINGCLOBF
    --------------------------------------------------------------------------------
    [0,1.0E+000,0,2.0E+000,0]
    
    1 row selected.
    ```

**Parent topic:** [ Vector Serializers ](vector-serializers.md)
