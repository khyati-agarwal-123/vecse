## Optimizer Plans for IVF Vector Indexes {#GUID-D3DF07F9-F75C-4BD8-937E-94DD1072BED3}

Inverted File Flat (IVF) is a form of Neighbor Partition Vector index. It is a partition-based index that achieves search efficiency by narrowing the search area through the use of neighbor partitions or clusters.

When using IVF vector indexes, you can see two possible plans for your similarity searches:

  * *Pre-filtering* evaluates the filters first, before using the IVF vector index. Once the optimizer chooses which centroid partitions to scan, it evaluates the filters for all the rows in those partitions. Then it computes the distance to the query vector for the rows that passed the filters. 
  * *Post-filtering* evaluates the filters after using the IVF vector index. Once the optimizer chooses which centroid partitions to scan, it computes the distance to the query vector for all the rows in those partitions. Once the optimizer finds the closest rows, it then evaluates the filter for those rows and returns only the ones that pass the filters. 



Consider the following query:
```
    SELECT chunk_id, chunk_data,
    FROM doc_chunks
    WHERE doc_id=1
    ORDER BY VECTOR_DISTANCE( chunk_embedding, :query_vector, COSINE )
    FETCH APPROX FIRST 4 ROWS ONLY WITH TARGET ACCURACY 80;
```
    

The preceding query can lead to the following two different execution plans:

> **note:** Depending on the version you are using, you can find variations of these plans. However, the idea stays the same, so the following plans are just for illustration purposes. 

Pre-filter plan

  * Plan line ids 5 to 9: This part happens first. The optimizer chooses the centroid `ids` that are closest to the query vector. 
  * Plan line ids 10 to 13: With these lines, the base table is joined to the identified centroid partitions using a `NESTED LOOPS` join. The base table is scanned to look for all the rows that pass the `WHERE` clause filter. For each of these rows, there is a look up in the index of the centroid partitions (plan line id 13). The `rowid` from the scan of the base table is used to find the row in the centroid partitions table that has a matching value for the base table `rowid` column. 
  * Plan line id 4: All the rows from 5-9 are joined to the rows from 10-13 with a `HASH JOIN`. That is, the rows that were filtered by 10-13 are reduced to only the rows from the partitions that were chosen as the closest ones. 
  * Plan line id 3: This step computes the vector distance and sorts on this value, only keeping the top-K rows.
```
    -------------------------------------------------------------------------------------------------------------------
    | Id  | Operation                               | Name                                                            |
    -------------------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT                        |                                                                 |
    |*  1 |  COUNT STOPKEY                          |                                                                 |
    |   2 |   VIEW                                  |                                                                 |
    |*  3 |    SORT ORDER BY STOPKEY                |                                                                 |
    |*  4 |     HASH JOIN                           |                                                                 |
    |   5 |      VIEW                               | VW_IVCR_2D77159E                                                |
    |*  6 |       COUNT STOPKEY                     |                                                                 |
    |   7 |        VIEW                             | VW_IVCN_9A1D2119                                                |
    |*  8 |         SORT ORDER BY STOPKEY           |                                                                 |
    |   9 |          TABLE ACCESS FULL              | VECTOR$DOCS_IVF_IDX2$81357_82648_0$IVF_FLAT_CENTROIDS           |
    |  10 |      NESTED LOOPS                       |                                                                 |
    |* 11 |       TABLE ACCESS FULL                 | DOC_CHUNKS                                                      |
    |  12 |       TABLE ACCESS BY GLOBAL INDEX ROWID| VECTOR$DOCS_IVF_IDX2$81357_82648_0$IVF_FLAT_CENTROID_PARTITIONS |
    |* 13 |        INDEX UNIQUE SCAN                | SYS_C008661                                                     |
    -------------------------------------------------------------------------------------------------------------------
```
    

> **note:** IVF indexes use auxiliary partition tables and associated indexes to store index information. Mainly `VECTOR$<index name>$<ids>$IVF_FLAT_CENTROIDS` and `VECTOR$<index name>$<ids>$IVF_FLAT_CENTROIDS_PARTITIONS`. 

Post-filter Plan

  * Plan line ids 11 to 15: Here the optimizer chooses the closest centroids (similar to lines 5 through 9 of the pre-filter plan).
  * Plan line id 10: This creates a special filter with information about which centroid `ids` were chosen. 
  * Plan line ids 16 and 17: These lines use the special filter from plan line 10 to scan only the corresponding centroid partitions.
  * Plan line id 9: This `HASH JOIN` makes sure that only the rows with the closest centroid `ids` are returned. 
  * Plan line id 8: This computes the distance for those rows with the closest centroid `ids` and finds the top-K rows. 
  * Plan line id 4: This `NESTED LOOPS` applies the base table filter to the previously identified top-K rows. 
```
    ---------------------------------------------------------------------------------------------------------------
    | Id  | Operation                           | Name                                                            |
    ---------------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT                    |                                                                 |
    |*  1 |  COUNT STOPKEY                      |                                                                 |
    |   2 |   VIEW                              |                                                                 |
    |*  3 |    SORT ORDER BY STOPKEY            |                                                                 |
    |   4 |     NESTED LOOPS                    |                                                                 |
    |   5 |      VIEW                           | VW_IVPSR_11E7D7DE                                               |
    |*  6 |       COUNT STOPKEY                 |                                                                 |
    |   7 |        VIEW                         | VW_IVPSJ_578B79F1                                               |
    |*  8 |         SORT ORDER BY STOPKEY       |                                                                 |
    |*  9 |          HASH JOIN                  |                                                                 |
    |  10 |           PART JOIN FILTER CREATE   | :BF0000                                                         |
    |  11 |            VIEW                     | VW_IVCR_B5B87E67                                                |
    |* 12 |             COUNT STOPKEY           |                                                                 |
    |  13 |              VIEW                   | VW_IVCN_9A1D2119                                                |
    |* 14 |               SORT ORDER BY STOPKEY |                                                                 |
    |  15 |                TABLE ACCESS FULL    | VECTOR$DOCS_IVF_IDX4$81357_83292_0$IVF_FLAT_CENTROIDS           |
    |  16 |           PARTITION LIST JOIN-FILTER|                                                                 |
    |  17 |            TABLE ACCESS FULL        | VECTOR$DOCS_IVF_IDX4$81357_83292_0$IVF_FLAT_CENTROID_PARTITIONS |
    |* 18 |      TABLE ACCESS BY USER ROWID     | DOC_CHUNKS                                                      |
    ---------------------------------------------------------------------------------------------------------------
```
    

IVF Indexes in the Optimizer Plan

If the IVF index is used by the optimizer, the plan contains the names of the centroids and centroid partitions tables accessed as well as corresponding indexes.

The value displayed in the `Options` column for tables accessed via IVF indexes depends upon whether the table scan is for a regular table or Exadata table. 

**Table: Centroids and Centroid Partition Table Options**

Operation | Options | Object_name  
---|---|---  
TABLE ACCESS | FULL |  `VECTOR$$<id>$IVF_FLAT_CENTROIDS`<br>`VECTOR$DOCS_IVF_IDX2$<id>$IVF_FLAT_CENTROID_PARTITIONS`  
TABLE ACCESS | STORAGE FULL |  `VECTOR$<vector-index-name>$<id>$IVF_FLAT_CENTROIDS`<br>`VECTOR$DOCS_IVF_IDX2$<id>$IVF_FLAT_CENTROID_PARTITIONS`  
TABLE ACCESS | INMEMORY FULL |  `VECTOR$$<id>$IVF_FLAT_CENTROIDS`<br>`VECTOR$DOCS_IVF_IDX2$<id>$IVF_FLAT_CENTROID_PARTITIONS`  
  
**Parent topic:** [Optimizer Plans for Vector Indexes](optimizer-plans-vector-indexes.md)
