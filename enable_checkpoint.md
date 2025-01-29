## ENABLE_CHECKPOINT {#GUID-B6B44353-4AE5-4A3F-BD5F-2F014A642014}

Use the `ENABLE_CHECKPOINT` procedure to enable the Checkpoint feature for a given Hierarchical Navigable Small World (HNSW) index user and HNSW index name. 

> **note:** 

  * This procedure only allows the index to create checkpoints. The checkpoint is created as part of the next HNSW graph refresh.

  * By default, HNSW checkpointing is enabled. If required, you can disable it using the `DBMS_VECTOR.DISABLE_CHECKPOINT` procedure. 




Syntax
```
    DBMS_VECTOR.ENABLE_CHECKPOINT('INDEX_USER',['INDEX_NAME']);
```
    

INDEX_USER

Specify the user name of the HNSW vector index owner.

INDEX_NAME

Specify the name of the HNSW vector index for which you want to enable the Checkpoint feature.

The `INDEX_NAME` clause is optional. If you do not specify the index name, then this procedure enables the Checkpoint feature for all HNSW vector indexes under the given user. 

Examples

  * Using both the index name and index user:
```
    DBMS_VECTOR.ENABLE_CHECKPOINT('VECTOR_USER','VIDX1');
```
    

  * Using only the index user:
```
    DBMS_VECTOR.ENABLE_CHECKPOINT('VECTOR_USER');
```
    




**Related Topics**

  * [*Oracle Database AI Vector Search User's Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-8604A7A5-3C96-4B55-85BC-BCF44562BDBB)
  * [DISABLE_CHECKPOINT](disable_checkpoint.md#GUID-BD7B95F2-5D35-4D94-9A34-ECE37D067C73)



**Parent topic:** [DBMS_VECTOR](dbms_vector-vecse.md)
