## $VECTORS {#GUID-2B53FFD8-D2A4-469D-83B1-C653602FFAEE}

This dictionary table provides information about the vector index part of a hybrid vector index, showing the contents of the `$VR` table with row ids, chunks, and embeddings. 

> **note:** The `*``*$VECTORS` view resides in a user's schema. Therefore, if a hybrid vector index is named `IDX`, then the view name is `IDX$VECTORS`. 

`*``*$VECTORS` view is not supported for Local Hybrid Vector Indexes. 

Column Name | Data Type | Description  
---|---|---  
`DOC_ROWID` |  `ROWID` |  Document table's row ID, excluding lazy deletes  
`DOC_CHUNK_ID` |  `NUMBER` |  ID for each chunk text  
`DOC_CHUNK_COUNT` |  `NUMBER` |  Number of chunks into which each document is split  
`DOC_CHUNK_OFFSET` |  `NUMBER` |  Original position of each chunk in the source document, relative to the start of document (which has a position of 1)  
`DOC_CHUNK_LENGTH ` |  `NUMBER` |  Character length of each chunk text  
`DOC_CHUNK_TEXT ` |  `VARCHAR2(4000)` |  Human-readable content in each chunk  
`DOC_CHUNK_EMBEDDING` |  `VECTOR(*, *)` |  Generated vector embedding for each chunk  
  
**Related Topics**

  * [Manage Hybrid Vector Indexes](manage-hybrid-vector-indexes.md#GUID-F2493927-23F8-4231-862B-6EDFA5A12299)



**Parent topic:** [Vector Index and Hybrid Vector Index Views](vector-index-and-hybrid-vector-index-views.md)
