## VECSYS.VECTOR$INDEX$CHECKPOINTS {#GUID-08AEA2D0-663A-4152-B6B9-750120EC6F87}

This dictionary table provides detailed information about Hierarchical Navigable Small World (HNSW) full checkpoints at the database level.

Column Name | Data Type | Description  
---|---|---  
`INDEX_OBJN` |  `NUMBER` |  The index object number to uniquely identify the HNSW index.  
`INDEX_OWNER_ID` |  `NUMBER` |  The owner id for the vector index.  
`CHECKPOINT_ID` |  `NUMBER` |  A monotonically increasing ID tracking various checkpoints for this HNSW index.  
`CHECKPOINT_SCN` |  `NUMBER` |  The SCN as of which the HNSW checkpoint is taken.   
`CHECKPOINT_TYPE` |  `NUMBER` |  Full Checkpoint is the only checkpoint type supported.  
`VERSION_NUMBER` |  `NUMBER` |  The checkpoint format may change in between releases. Thus, you can use a version number to decide whether to reload the HNSW index using a particular checkpoint.  
`TABLESPACE_NUMBER` |  `NUMBER` |  The tablespace number. You can view a tablespace number to decide whether to store full checkpoints of your HNSW graphs in the same tablespace or in a different one.  
  
**Related Topics**

  * [Understand HNSW Index Population Mechanisms in Oracle RAC or Single Instance](understand-hnsw-index-population-mechanisms-oracle-rac-and-single-instance.md#GUID-8604A7A5-3C96-4B55-85BC-BCF44562BDBB)



**Parent topic:** [Vector Index and Hybrid Vector Index Views](vector-index-and-hybrid-vector-index-views.md)
