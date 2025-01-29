## INDEX_VECTOR_MEMORY_ADVISOR {#GUID-68009857-9F68-4894-8341-46EF81975A1B}

Use the `INDEX_VECTOR_MEMORY_ADVISOR` procedure to determine the vector memory size needed for a particular vector index. This helps you evaluate the number of indexes that can fit for each simulated vector memory size. 

Syntax

  * Using the number and type of vector dimensions that you want to store in your vector index. 
```
    DBMS_VECTOR.INDEX_VECTOR_MEMORY_ADVISOR(
    INDEX_TYPE,
    NUM_VECTORS,
    DIM_COUNT,
    DIM_TYPE,
    PARAMETER_JSON,
    RESPONSE_JSON);
```
    

  * Using the table and vector column on which you want to create your vector index: 
```
    DBMS_VECTOR.INDEX_VECTOR_MEMORY_ADVISOR(
    TABLE_OWNER,
    TABLE_NAME,
    COLUMN_NAME,
    INDEX_TYPE,
    PARAMETER_JSON,
    RESPONSE_JSON);
```
    




**Table: Syntax Details: INDEX_VECTOR_MEMORY_ADVISOR**

Parameter | Description  
---|---  
`INDEX_TYPE` |  Type of vector index: <br>* `IVF` for Inverted File Flat (IVF) vector indexes  <br>`HNSW` for Hierarchical Navigable Small World (HNSW) vector indexes   
  
`NUM_VECTORS` |  Number of vectors that you plan to create the vector index with.  
`DIM_COUNT` |  Number of dimensions of a vector as a `NUMBER`.   
`DIM_TYPE` |  Type of dimensions of a vector. Possible values are: <br>* `FLOAT16` <br>`FLOAT32` <br>`FLOAT64` <br>`INT8`  
  
`TABLE_OWNER` |  Owner name of the table on which to create the vector index.  
`TABLE_NAME` |  Table name on which to create the vector index.  
`COLUMN_NAME` |  Name of the vector column on which to create the vector index.   
`PARAMETER_JSON` |  Input parameter in JSON format. You can specify only one of the following form: <br>* `PARAMTER_JSON=>{"accuracy": <br>value}` <br>`INDEX_TYPE=>IVF, parameter_json=>{"neighbor_partitions": <br>value}` <br>`INDEX_TYPE=>HNSW, parameter_json=>{"neighbors": <br>value}`
<br>**Note**: <br>You cannot specify values for `accuracy` along with `neighbor_partitions` or `neighbors`.   
`RESPONSE_JSON` |  JSON-formatted response string.  
  
Examples

  * Using neighbors in the parameters list: 
```
    exec DBMS_VECTOR.INDEX_VECTOR_MEMORY_ADVISOR(
    INDEX_TYPE=>'HNSW',
    NUM_VECTORS=>10000,
    DIM_COUNT=>768,
    DIM_TYPE=>'FLOAT32',
    PARAMETER_JSON=>'{"neighbors":128}',
    RESPONSE_JSON=>:response_json);
    
    Suggested vector memory pool size: 59918628 Bytes
```
    

  * Using accuracy in the parameters list:
```
    exec DBMS_VECTOR.INDEX_VECTOR_MEMORY_ADVISOR(
    INDEX_TYPE=>'HNSW',
    NUM_VECTORS=>10000,
    DIM_COUNT=>768,
    DIM_TYPE=>'FLOAT32',
    PARAMETER_JSON=>'{"accuracy":90}',
    RESPONSE_JSON=>:response_json);
    
    Suggested vector memory pool size: 53926765 Bytes
```
    

  * Using the table and vector column on which you want to create the vector index:
```
    exec DBMS_VECTOR.INDEX_VECTOR_MEMORY_ADVISOR(
    'VECTOR_USER',
    'VECTAB',
    'DATA_VECTOR',
    'HNSW',
    RESPONSE_JSON=>:response_json);
    
    Using default accuracy: 90%
    Suggested vector memory pool size: 76396251 Bytes
```
    




**Related Topics**

  * [*Oracle Database AI Vector Search User's Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-5D9B6B92-C62C-4927-9FB2-7A4437F24A19)



**Parent topic:** [DBMS_VECTOR](dbms_vector-vecse.md)
