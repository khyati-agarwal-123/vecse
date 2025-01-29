## QUERY {#GUID-BCB1DEE6-E8E0-43EA-9E5F-E346C981BC05}

Use the `DBMS_VECTOR.QUERY` function to perform a similarity search operation which returns the top-k results as a JSON array. 

Syntax

Query is overloaded and supports a version with `query_vector` passed in as a `VECTOR` type in addition to `CLOB`. 
```
    DBMS_VECTOR.QUERY (
    TAB_NAME             IN VARCHAR2,
    VEC_COL_NAME         IN VARCHAR2,
    QUERY_VECTOR         IN CLOB,
    TOP_K                IN NUMBER,
    VEC_PROJ_COLS        IN JSON_ARRAY_T DEFAULT NULL,
    IDX_NAME             IN VARCHAR2 DEFAULT NULL,
    DISTANCE_METRIC      IN VARCHAR2 DEFAULT 'COSINE',
    USE_INDEX            IN BOOLEAN DEFAULT TRUE,
    ACCURACY             IN NUMBER DEFAULT '90',
    IDX_PARAMETERS       IN CLOB DEFAULT NULL
    ) return JSON_ARRAY_T;
    
    DBMS_VECTOR.QUERY (
    TAB_NAME             IN VARCHAR2,
    VEC_COL_NAME         IN VARCHAR2,
    QUERY_VECTOR         IN VECTOR,
    TOP_K                IN NUMBER,
    VEC_PROJ_COLS        IN JSON_ARRAY_T DEFAULT NULL,
    IDX_NAME             IN VARCHAR2 DEFAULT NULL,
    DISTANCE_METRIC      IN VARCHAR2 DEFAULT 'COSINE',
    USE_INDEX            IN BOOLEAN DEFAULT TRUE,
    ACCURACY             IN NUMBER DEFAULT '90',
    IDX_PARAMETERS       IN CLOB DEFAULT NULL
    ) return JSON_ARRAY_T;
```
    

Parameters

Specify the input parameters in JSON format.

**Table: DBMS_VECTOR.QUERY Parameters**

Parameter | Description  
---|---  
`tab_name` |  Table name to query  
`vec_col_name` |  Vector column name   
`query_vector` |  Query vector passed in as `CLOB` or `VECTOR`.   
`top_k` |  Number of results to be returned.  
`vec_proj_cols` |  Columns to be projected as part of the result.  
`idx_name` |  Name of the index queried.  
`distance_metric` |  Distance computation metric. Defaults to `COSINE`. Can also be `MANHATTAN`, `HAMMING`, `DOT`, `EUCLIDEAN`, `L2_SQUARED`, `EUCLIDEAN_SQUARED`. .   
`use_index` |  Specifies whether the search is an approximate search or exact search. Defaults to TRUE (that is, approximate).  
`accuracy` |  Specifies the minimum desired query accuracy.  
`idx_parameters` |  Specifies values of `efsearch` and `neighbor partition probes` passed in, formatted as JSON   
  
DATA

This function accepts the input data type as `VARCHAR2`, `NUMBER`, `JSON`, `BOOLEAN` or `CLOB`. 

**Parent topic:** [DBMS_VECTOR](dbms_vector-vecse.md)
