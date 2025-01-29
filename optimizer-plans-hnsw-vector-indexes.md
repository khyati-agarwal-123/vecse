## Optimizer Plans for HNSW Vector Indexes {#GUID-7D6B60D4-5A0C-4E9F-963E-81E244F2847A}

A Hierarchical Navigable Small World Graph (HNSW) is a form of In-Memory Neighbor Graph vector index. It is a very efficient index for vector approximate similarity search.

In the simplest case, a query has a single table and it does not contain any relational filter predicates or subqueries. The following query example illustrates this situation: 
```
    SELECT chunk_id, chunk_data
    FROM doc_chunks
    ORDER BY VECTOR_DISTANCE( chunk_embedding, :query_vector, COSINE )
    FETCH APPROX FIRST 4 ROWS ONLY WITH TARGET ACCURACY 80;
```
    

The corresponding execution plan should look like the following, if the optimizer decides to use the index (start from operation id 5):

Top-K similar vectors are first identified using the HNSW vector index. For each of them, corresponding rows are identified in the base table and required selected columns are extracted.
```
    ---------------------------------------------------------
    | Id  | Operation                      | Name           |
    ---------------------------------------------------------
    |   0 | SELECT STATEMENT               |                |
    |*  1 |  COUNT STOPKEY                 |                |
    |   2 |   VIEW                         |                |
    |*  3 |    SORT ORDER BY STOPKEY       |                |
    |   4 |     TABLE ACCESS BY INDEX ROWID| DOC_CHUNKS     |
    |   5 |      VECTOR INDEX HNSW SCAN    | DOCS_HNSW_IDX5 |
    ---------------------------------------------------------
```
    

> **note:** The Hierarchical Navigable Small World (HNSW) vector index structure contains `rowids` of corresponding base table rows for each vector in the index. 

However, your query may contain traditional relational data filters, as illustrated in the following example:
```
    SELECT chunk_id, chunk_data
    FROM doc_chunks
    WHERE doc_id=1
    ORDER BY VECTOR_DISTANCE( chunk_embedding, :query_vector, COSINE )
    FETCH APPROX FIRST 4 ROWS ONLY WITH TARGET ACCURACY 80;
```
    

In that case, there are essentially two main alternatives for the optimizer to generate an execution plan using an HNSW vector index. These two alternatives are called **pre-filtering** and **in-filtering**. 

The main difference between pre-filtering and in-filtering are:

  * *Pre-filtering* runs the filtering evaluation on the base table first and only traverses the HNSW vector index for corresponding vectors. This can be very fast if the filter predicates are selective (that is, most rows filtered out). 
  * *In-filtering*, on the other hand, starts by traversing the HNSW vector index and invokes the filtering only for each vector matching candidate. This can be better than pre-filtering when more rows pass the filter predicates. In this case, the number of vector candidates to consider, while traversing the HNSW vector index, might be many fewer than the number of rows that pass the filters. 



For both in-filtering and pre-filtering, the optimizer may choose to process projected columns from your select list before or after the similarity search operation. If it does so after, this is called a join-back operation. If it does so before, it is called a no-join-back operation. The tradeoff between the two depends on the number of rows returned by the similarity search.

To illustrate these four possibilities, here are some typical execution plans:

> **note:** Depending on the version you are using, you can find variations of these plans. However, the idea stays the same, so the following plans are just for illustration purposes. 

Pre-filter with Join-back (Start from Operation id 9)

Filtered `rowids` are first identified in the base table and joined to the auxiliary mapping table before the HNSW vector index identifies the relevant vectors. Relevant base table rows are then identified and required selected columns are extracted. 
```
    --------------------------------------------------------------------------------------------------------
    | Id  | Operation                             | Name                                                   |
    --------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT                      |                                                        |
    |*  1 |  COUNT STOPKEY                        |                                                        |
    |   2 |   VIEW                                |                                                        |
    |*  3 |    SORT ORDER BY STOPKEY              |                                                        |
    |*  4 |     TABLE ACCESS BY INDEX ROWID       | DOC_CHUNKS                                             |
    |   5 |      VECTOR INDEX HNSW SCAN PRE-FILTER| DOCS_HNSW_IDX3                                         |
    |   6 |       VIEW                            | VW_HPJ_56DFF779                                        |
    |*  7 |        HASH JOIN RIGHT OUTER          |                                                        |
    |   8 |         TABLE ACCESS FULL             | VECTOR$DOCS_HNSW_IDX3$81357_82726_0$HNSW_ROWID_VID_MAP |
    |*  9 |         TABLE ACCESS FULL             | DOC_CHUNKS                                             |
    --------------------------------------------------------------------------------------------------------
```
    

> **note:** 

  * HNSW indexes use an auxiliary table and associated index to store index information like mapping between vector `ids` and `rowids`. Mainly `VECTOR$<index name>$HNSW_ROWID_VID_MAP`. 
  * For the join, you can also see plans like the following:
```
    NESTED LOOPS OUTER
    TABLE ACCESS FULL  DOC_CHUNKS
    TABLE ACCESS BY INDEX ROWID VECTOR$DOCS_HNSW_IDX3$81357_82726_0$HNSW_ROWID_VID_MAP
    INDEX UNIQUE SCAN SYS_C008586
```
    




Pre-filter without Join-back (Start from Operation id 8)

Filtered `rowids` with relevant columns are first identified in the base table and buffered. The result is then joined to the auxiliary mapping table before the HNSW vector index identifies the top-K vectors. For those identified vectors, associated base table columns are shown. 
```
    -------------------------------------------------------------------------------------------------------
    | Id  | Operation                            | Name                                                   |
    -------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT                     |                                                        |
    |*  1 |  COUNT STOPKEY                       |                                                        |
    |   2 |   VIEW                               |                                                        |
    |*  3 |    SORT ORDER BY STOPKEY             |                                                        |
    |   4 |     VECTOR INDEX HNSW SCAN PRE-FILTER| DOCS_HNSW_IDX3                                         |
    |   5 |      VIEW                            | VW_HPF_B919B0A0                                        |
    |*  6 |       HASH JOIN RIGHT OUTER          |                                                        |
    |   7 |        TABLE ACCESS FULL             | VECTOR$DOCS_HNSW_IDX3$81357_82726_0$HNSW_ROWID_VID_MAP |
    |*  8 |        TABLE ACCESS FULL             | DOC_CHUNKS                                             |
    -------------------------------------------------------------------------------------------------------
```
    

In-filter With Join-back (Start from Operation id 5)

HNSW vector index is traversed first, and for each identified vector, filters on the base table for the corresponding `rowid` are applied. Once the top-K `rowids` passing the filters are identified, base table columns are extracted. 
```
    ----------------------------------------------------------------
    | Id  | Operation                            | Name            |
    ----------------------------------------------------------------
    |   0 | SELECT STATEMENT                     |                 |
    |*  1 |  COUNT STOPKEY                       |                 |
    |   2 |   VIEW                               |                 |
    |*  3 |    SORT ORDER BY STOPKEY             |                 |
    |*  4 |     TABLE ACCESS BY INDEX ROWID      | DOC_CHUNKS      |
    |   5 |      VECTOR INDEX HNSW SCAN IN-FILTER| DOCS_HNSW_IDX3  |
    |   6 |       VIEW                           | VW_HIJ_B919B0A0 |
    |*  7 |        TABLE ACCESS BY USER ROWID    | DOC_CHUNKS      |
    ----------------------------------------------------------------
```
    

In-filter without Join-back (Start from Operation id 4)

HNSW vector index is traversed first, and for each identified vector, filters on the base table for the corresponding `rowid` are applied and relevant columns extracted. Once the top-K vectors are identified, corresponding base table columns are shown. 
```
    ---------------------------------------------------------------
    | Id  | Operation                           | Name            |
    ---------------------------------------------------------------
    |   0 | SELECT STATEMENT                    |                 |
    |*  1 |  COUNT STOPKEY                      |                 |
    |   2 |   VIEW                              |                 |
    |*  3 |    SORT ORDER BY STOPKEY            |                 |
    |   4 |     VECTOR INDEX HNSW SCAN IN-FILTER| DOCS_HNSW_IDX3  |
    |   5 |      VIEW                           | VW_HIF_B919B0A0 |
    |*  6 |       TABLE ACCESS BY USER ROWID    | DOC_CHUNKS      |
    ---------------------------------------------------------------
```
    

HNSW Indexes in the Optimizer Plan

HNSW indexes plans may use an internal table and associated index to store index information like mapping between vector `ids` and `rowids`. Mainly `VECTOR$<index name>$HNSW_ROWID_VID_MAP`. 

**Table: HNSW Options**

Operation | Options | Object_name  
---|---|---  
TABLE ACCESS | FULL | `VECTOR$<vector-index-name>$<id>$HNSW_ROW_VID_MAP`  
TABLE ACCESS | STORAGE FULL | `VECTOR$<vector-index-name>$<id>$HNSW_ROW_VID_MAP`  
TABLE ACCESS | INMEMORY FULL | `VECTOR$<vector-index-name>$<id>$HNSW_ROW_VID_MAP`  
  
For the HNSW index itself, which is an In-Memory object, the plan uses a `VECTOR INDEX` operation. The object name `GALAXIES_HNSW_INX` is provided as an example of the user-specified HNSW index name: 

Operation  | Options | Object_name  
---|---|---  
VECTOR INDEX | HNSW SCAN | `GALAXIES_HNSW_INX`  
VECTOR INDEX | HNSW SCAN PRE-FILTER | `GALAXIES_HNSW_INX`  
VECTOR INDEX | HNSW SCAN IN-FILTER | `GALAXIES_HNSW_INX`  
  
**Parent topic:** [Optimizer Plans for Vector Indexes](optimizer-plans-vector-indexes.md)
