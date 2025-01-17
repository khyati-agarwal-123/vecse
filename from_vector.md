## FROM_VECTOR {#GUID-AA60B3CB-FCB7-4944-9E06-976C272855B1}

`FROM_VECTOR` takes a vector as input and returns a string of type `VARCHAR2` or `CLOB` as output. 

Syntax

<br>![Description of from_vector.eps follows](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img/from_vector.gif)<br>[Descriptionof the illustration from_vector.eps](https://docs.oracle.com/en/database/oracle/oracle-database/23/vecse/img_text/from_vector.md)

<br>Purpose<br>`FROM_VECTOR` optionally takes a `RETURNING` clause to specify the data type of the returned value. <br>If `VARCHAR2` is specified without size, the size of the returned value size is 32767. <br>You can optionally specify the text format of the output in the `FORMAT` clause, using the tokens `SPARSE` or `DENSE`. Note that the input vector storage format does not need to match the specified output format. <br>There is no support to convert to `CHAR`, `NCHAR`, and `NVARCHAR2`. <br>`FROM_VECTOR` is synonymous with `VECTOR_SERIALIZE`. 

Parameters<br>*expr* must evaluate to a vector. The function returns NULL if *expr* is NULL. 

Examples
```
SELECT FROM_VECTOR(TO_VECTOR('[1, 2, 3]') );


FROM_VECTOR(TO_VECTOR('[1,2,3]'))
–---------------------------------------------------------------
[1.0E+000,2.0E+000,3.0E+000]

1 row selected.
```
```
SELECT FROM_VECTOR(TO_VECTOR('[1.1, 2.2, 3.3]', 3, FLOAT32) );


FROM_VECTOR(TO_VECTOR('[1.1,2.2,3.3]',3,FLOAT32))
------------------------------------------------------------------
[1.10000002E+000,2.20000005E+000,3.29999995E+000]

1 row selected.
```
```
SELECT FROM_VECTOR( TO_VECTOR('[1.1, 2.2, 3.3]', 3, FLOAT32) RETURNING VARCHAR2(1000));


FROM_VECTOR(TO_VECTOR('[1.1,2.2,3.3]',3,FLOAT32)RETURNINGVARCHAR2(1000))
--------------------------------------------------------------------------------
[1.10000002E+000,2.20000005E+000,3.29999995E+000]

1 row selected.
```
```
SELECT FROM_VECTOR(TO_VECTOR('[1.1, 2.2, 3.3]', 3, FLOAT32) RETURNING CLOB );


FROM_VECTOR(TO_VECTOR('[1.1,2.2,3.3]',3,FLOAT32)RETURNINGCLOB)
--------------------------------------------------------------------------------
[1.10000002E+000,2.20000005E+000,3.29999995E+000]

1 row selected.
```
```
SELECT FROM_VECTOR(TO_VECTOR('[5,[2,4],[1.0,2.0]]', 5, FLOAT64, SPARSE) RETURNING CLOB FORMAT SPARSE);

FROM_VECTOR(TO_VECTOR('[5,[2,4],[1.0,2.0]]',5,FLOAT64,SPARSE)RETURNINGCLOBFORMAT
--------------------------------------------------------------------------------
[5,[2,4],[1.0E+000,2.0E+000]]

1 row selected.
```
```
SELECT FROM_VECTOR(TO_VECTOR('[5,[2,4],[1.0,2.0]]', 5, FLOAT64, SPARSE) RETURNING CLOB FORMAT DENSE);

FROM_VECTOR(TO_VECTOR('[5,[2,4],[1.0,2.0]]',5,FLOAT64,SPARSE)RETURNINGCLOBFORMAT
--------------------------------------------------------------------------------
[0,1.0E+000,0,2.0E+000,0]

1 row selected.
```


> **note:** 

* Applications using Oracle Client 23ai libraries or Thin mode drivers can fetch vector data directly, as shown in the following example:
```
SELECT dataVec FROM vecTab;
```


* For applications using Oracle Client 23ai libraries prior to 23ai connected to Oracle Database 23ai, use the `FROM_VECTOR` to fetch vector data, as shown by the following example: 
```
SELECT FROM_VECTOR(dataVec) FROM vecTab;
```





**Parent topic:** [Vector Serializers](vector-serializers.md)