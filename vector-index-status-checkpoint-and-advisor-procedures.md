## Vector Index Status, Checkpoint, and Advisor Procedures {#GUID-5F1D700B-ACF0-4E3A-A9E5-00B65B4CCF3C}

Review these high-level details on the `GET_INDEX_STATUS`, `ENABLE_CHECKPOINT`, `DISABLE_CHECKPOINT`, and `INDEX_VECTOR_MEMORY_ADVISOR` procedures that are available with the `DBMS_VECTOR` PL/SQL package. 

GET_INDEX_STATUS

**Purpose**: To query the status of a vector index creation. 

**Syntax**: 
```
    DBMS_VECTOR.GET_INDEX_STATUS ('USER_NAME','INDEX_NAME');
```
    

**Usage Notes**: 

  * You can use the `GET_INDEX_STATUS` procedure only during a vector index creation. 

  * The `Percentage` value is shown in the output only for Hierarchical Navigable Small World (HNSW) indexes (and not for Inverted File Flat (IVF) indexes). 

  * Along with the `DB_DEVELOPER_ROLE` privilege, you must have read access to the `VECSYS.VECTOR$INDEX$BUILD$` table. 

  * You can use the following query to view all auxiliary tables: 
```
    select IDX_AUXILIARY_TABLES from vecsys.vector$index;
```
    

    * For HNSW indexes: 

`rowid_vid_map` stores the mapping between a row ID and vector ID. `shared_journal_change_log` stores the DML changes that are yet to be incorporated into an HNSW graph. 

    * For IVF indexes: 

`centroids` stores the location for each centroid. `centroid_partitions` stores the best centroid for each vector. 

  * The possible values of `Stage` for HNSW vector indexes are: 

Value | Description  
---|---  
`HNSW Index Initialization` |  Initialization phase for the HNSW vector index creation  
`HNSW Index Auxiliary Tables Creation` |  Creation of the internal auxiliary tables for the HNSW Neighbor Graph vector index  
`HNSW Index Graph Allocation` |  Allocation of memory from the vector memory pool for the HNSW graph  
`HNSW Index Loading Vectors` |  Loading of the base table vectors into the vector pool memory   
`HNSW Index Graph Construction` |  Creation of the multi-layered HNSW graph with the previously loaded vectors  
`HNSW Index Creation Completed` |  HNSW vector index creation finished  
  
  * The possible values of `Stage` for IVF vector indexes are: 

Value | Description  
---|---  
`IVF Index Initialization` |  Initialization phase for the IVF vector index creation  
`IVF Index Centroids Creation` |  The K-means clustering phase that computes the cluster centroids on a sample of base table vectors  
`IVF Index Centroid Partitions Creation` |  Centroids assignment phase for the base table vectors  
`IVF Index Creation Completed` |  IVF vector index creation completed  
  



**Example**: 
```
    exec DBMS_VECTOR.GET_INDEX_STATUS('VECTOR_USER','VIDX_HNSW');
    
    Index objn: 74745
    Stage: HNSW Index Loading Vectors
    Percentage: 80%
```
    

ENABLE_CHECKPOINT

**Purpose**: To enable the Checkpoint feature for a given HNSW index user and HNSW index name. 

> **note:** This procedure only allows the index to create checkpoints. The checkpoint is created as part of the next HNSW graph refresh. 

The `INDEX_NAME` clause is optional. If you do not specify the index name, then this procedure enables the Checkpoint feature for all HNSW indexes under the given user. 

**Syntax**: 
```
    DBMS_VECTOR.ENABLE_CHECKPOINT('INDEX_USER',['INDEX_NAME']);
```
    

**Example 1: Using index name and index user**: 
```
    DBMS_VECTOR.ENABLE_CHECKPOINT('VECTOR_USER','VIDX1');
```
    

**Example 2: Using index user**: 
```
    DBMS_VECTOR.ENABLE_CHECKPOINT('VECTOR_USER');
```
    

> **note:** By default, HNSW checkpointing is enabled. You can disable it using the `DBMS_VECTOR.DISABLE_CHECKPOINT` procedure. 

DISABLE_CHECKPOINT

**Purpose**: To purge all older checkpoint data. This procedure disables the Checkpoint feature for a given HNSW index user and HNSW index name. It also disables the creation of future checkpoints as part of the HNSW graph refresh. 

The *`INDEX_NAME`* clause is optional. If you do not specify the index name, then this procedure disables the Checkpoint feature for all HNSW indexes under the given user. 

**Syntax**: 
```
    DBMS_VECTOR.DISABLE_CHECKPOINT('INDEX_USER',['INDEX_NAME']);
```
    

**Example 1: Using index name and index user**: 
```
    DBMS_VECTOR.DISABLE_CHECKPOINT('VECTOR_USER','VIDX1');
```
    

**Example 2: Using index user**: 
```
    DBMS_VECTOR.DISABLE_CHECKPOINT('VECTOR_USER');
```
    

INDEX_VECTOR_MEMORY_ADVISOR

**Purpose**: To determine the vector memory size needed for a particular vector index. This helps you evaluate the number of indexes (HNSW or IVF) that can fit for each simulated vector memory size. 

**Syntax**: 

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
    




`INDEX_TYPE` can be one of the following: 

  * `IVF` for IVF vector indexes 

  * `HNSW` for HNSW vector indexes 




`PARAMETER_JSON` can have only one of the following form: 

  * `PARAMTER_JSON=>{"accuracy":value}`

  * `INDEX_TYPE=>IVF, parameter_json=>{"neighbor_partitions":value}`

  * `INDEX_TYPE=>HNSW, parameter_json=>{"neighbors":value}`




> **note:** You cannot specify values for `accuracy` along with `neighbor_partitions` or `neighbors`. 

**Example 1: Using neighbors in the parameters list**: 
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
    

**Example 2: Using accuracy in the parameters list**: 
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
    

**Example 3: Using the table and vector column on which you want to create the vector index**: 
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

  * [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12)



**Parent topic:** [Manage the Different Categories of Vector Indexes](manage-different-categories-vector-indexes.md)
