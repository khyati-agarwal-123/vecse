## Oracle AI Vector Search Parameters {#GUID-5B240ECF-DAF1-44F9-8B61-37687580872B}

This is a set of parameters related to Oracle AI Vector Search.

  * **VECTOR_MEMORY_SIZE**

Syntax: `VECTOR_MEMORY_SIZE = [ON | OFF]` (default `ON`) 

The initialization parameter `VECTOR_MEMORY_SIZE` specifies either the current size of the Vector Pool (at CDB level) or the maximum Vector Pool usage allowed by a PDB (at PDB level). 

For more information about this parameter, see [Size the Vector Pool](size-vector-pool.md#GUID-1815E227-56C9-4E62-977F-0FDA282C9D83). 

  * **VECTOR_INDEX_NEIGHBOR_GRAPH_RELOAD**

Syntax: `VECTOR_INDEX_NEIGHBOR_GRAPH_RELOAD = [RESTART | OFF]` (default `RESTART`) 

The initialization parameter `VECTOR_INDEX_NEIGHBOR_GRAPH_RELOAD` is used to manage automatic recreation of HNSW indexes. You can enable or disable both the HNSW duplication and reload mechanisms in Oracle RAC and non-RAC environments. 

For more information about this parameter and the duplication and reload mechanisms, see [Understand HNSW Index Population Mechanisms in Oracle RAC or Single Instance](understand-hnsw-index-population-mechanisms-oracle-rac-and-single-instance.md#GUID-8604A7A5-3C96-4B55-85BC-BCF44562BDBB). 

  * **VECTOR_QUERY_CAPTURE**

Syntax: `VECTOR_QUERY_CAPTURE = [ON | OFF]` (default `ON`) 

The initialization parameter `VECTOR_QUERY_CAPTURE` is used to enable or disable capture of query vectors. 

For more information about this parameter and about capturing query vectors, see [Index Accuracy Report](index-accuracy-report.md#GUID-A084929C-7A04-4764-9E5B-1204E0844CAF). 




**Parent topic:** [Vector Diagnostics](vector-diagnostics-node.md)
