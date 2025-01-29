## Vector Index and Hybrid Vector Index Views {#GUID-F536133E-5B76-4866-BC91-59DD75EF90FA}

These views allow you to query tables related to vector indexes and hybrid vector indexes.

  * [VECSYS.VECTOR$INDEX](vecsys-vectorindex.md)  
This dictionary table contains detailed information about vector indexes. 
  * [V$VECTOR_INDEX](vvector_index.md)  
This fixed view provides diagnostic information about vector indexes and is available with Autonomous Database Serverless (ADB-S). 
  * [V$VECTOR_GRAPH_INDEX](vvector_graph_index.md)  
This fixed view provides diagnostic information about In-Memory Neighbor Graph vector indexes and is available with Autonomous Database Serverless (ADB-S). 
  * [V$VECTOR_PARTITIONS_INDEX](vvector_partitions_index.md)  
This fixed view provides diagnostic information about Inverted File Flat indexes and is available with Autonomous Database Serverless (ADB-S). 
  * [VECSYS.VECTOR$INDEX$CHECKPOINTS](vecsys-vectorindexcheckpoints.md)  
This dictionary table provides detailed information about Hierarchical Navigable Small World (HNSW) full checkpoints at the database level. 
  * [$VECTORS](indexnamevectors.md)  
This dictionary table provides information about the vector index part of a hybrid vector index, showing the contents of the `$VR` table with row ids, chunks, and embeddings. 



**Parent topic:** [Oracle AI Vector Search Views](oracle-ai-vector-search-views.md)
