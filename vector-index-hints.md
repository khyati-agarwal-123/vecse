## Vector Index Hints {#GUID-11D70C9F-13C0-433E-A946-C754E8B36417}

If the optimizer does not choose your existing index while running your query and you still want that index to be used, you can rewrite your SQL statement to include vector index hints.

There are two types of vector index hints:

  * *Access path hints* apply when running a simple query block with a single table with no predicates and a valid HNSW vector index. This only applies to HNSW vector indexes. 
  * *Query transformation hints* cover all other cases, where query transformations are used. 



Syntax for Access Path Hints

The access path hints are modeled on the existing `INDEX` hints. For example: 
```
    VECTOR_INDEX_SCAN ( [ @ queryblock ] tablespec [ indexspec ])
    
    NO_VECTOR_INDEX_SCAN ( [ @ queryblock ] tablespec [ indexspec ])
```
    

> **note:** See Also: 

  * [*Oracle Database SQL Language Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SQLRF-GUID-EC0D9F8A-20E7-4281-A16A-6B9C993F2930) for details about the `INDEX` hint 



Access Path Hint Examples
```
    SELECT /*+ VECTOR_INDEX_SCAN(galaxies) */ name
    FROM galaxies
    ORDER BY VECTOR_DISTANCE( embedding, to_vector('[0,1,1,0,0]'), COSINE )
    FETCH APPROXIMATE FIRST 3 ROWS ONLY;
```
```
    SELECT /*+ NO_VECTOR_INDEX_SCAN(galaxies) */ name
    FROM galaxies
    ORDER BY VECTOR_DISTANCE( embedding, to_vector('[0,1,1,0,0]'), COSINE )
    FETCH APPROXIMATE FIRST 3 ROWS ONLY;
```
    

Syntax for Query Transformation Hints

The syntax for query transformation hints is similar to the `INDEX` hints model. For example: 
```
    VECTOR_INDEX_TRANSFORM ( [ @ queryblock ] tablespec [ indexspec [ filtertype ]] )
    
    NO_VECTOR_INDEX_TRANSFORM ( [ @ queryblock ] tablespec [ indexspec ] )
```
    

`VECTOR_INDEX_TRANSFORM` forces the transformation and uses the optionally specified index, whereas `NO_VECTOR_INDEX_TRANSFORM` disables the transformation and the use of the optionally specified index. For more information about the potential query transformations, see [Optimizer Plans for HNSW Vector Indexes](optimizer-plans-hnsw-vector-indexes.md#GUID-7D6B60D4-5A0C-4E9F-963E-81E244F2847A) and [Optimizer Plans for IVF Vector Indexes](optimizer-plans-ivf-vector-indexes.md#GUID-D3DF07F9-F75C-4BD8-937E-94DD1072BED3). 

The optional `filtertype` specification for the `VECTOR_INDEX_TRANSFORM` hint can have the following values: 

  * `PRE_FILTER_WITH_JOIN_BACK` (only applies to HNSW indexes) 
  * `PRE_FILTER_WITHOUT_JOIN_BACK` (applies to both HNSW and IVF indexes) 
  * `IN_FILTER_WITH_JOIN_BACK` (only applies to HNSW indexes) 
  * `IN_FILTER_WITHOUT_JOIN_BACK` (only applies to HNSW indexes) 
  * `POST_FILTER_WITHOUT_JOIN_BACK` (only applies to IVF indexes) 



Each `filtertype` value corresponds to exactly one state in the query transformation. If no `filtertype` value is specified in the `VECTOR_INDEX_TRANSFORM` hint, then all of the valid filtering types are considered and the best one is chosen. 

> **note:** If you specify the `filtertype` value, you must also specify the `indexspec` value. 

> **note:** See Also: 

  * [*Oracle Database SQL Language Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SQLRF-GUID-EC0D9F8A-20E7-4281-A16A-6B9C993F2930) for details about the `INDEX` hint 



Query Transformation Hint Examples
```
    SELECT /*+ vector_index_transform(galaxies galaxies_ivf_idx pre_filter_without_join_back) */ name
    FROM galaxies
    WHERE id<5
    ORDER BY VECTOR_DISTANCE( embedding, to_vector('[0,1,1,0,0]'), COSINE )
    FETCH APPROXIMATE FIRST 3 ROWS ONLY WITH TARGET ACCURACY 90;
```
```
    SELECT /*+ vector_index_transform(galaxies) */ name
    FROM galaxies
    WHERE id<5
    ORDER BY VECTOR_DISTANCE( embedding, to_vector('[0,1,1,0,0]'), COSINE )
    FETCH APPROXIMATE FIRST 10 ROWS ONLY WITH TARGET ACCURACY 90;
```
```
    SELECT /*+ no_vector_index_transform(galaxies) */ name
    FROM galaxies
    WHERE id<5
    ORDER BY VECTOR_DISTANCE( embedding, to_vector('[0,1,1,0,0]'), COSINE )
    FETCH APPROXIMATE FIRST 10 ROWS ONLY WITH TARGET ACCURACY 90;
```
```
    SELECT /*+ vector_index_transform(galaxies galaxies_hnsw_idx in_filter_with_join_back) */ name
    FROM galaxies
    WHERE id<5
    ORDER BY VECTOR_DISTANCE( embedding, to_vector('[0,1,1,0,0]'), COSINE )
    FETCH APPROXIMATE FIRST 10 ROWS ONLY WITH TARGET ACCURACY 90;
```
    

> **note:** 

  * The auxiliary tables for vector indexes are regular heap tables and require up-to-date optimizer statistics for accurate costing. The automatic optimizer statistics collection task gathers statistics for the auxiliary tables only when collecting statistics on their base tables.
  * Using the `DBMS_STATS` package, you can perform manual statistics gathering operations on the auxiliary tables. However, `GATHER_INDEX_STATS` on vector indexes is not permitted. The in-memory HNSW index automatically maintains its statistics in-memory. 



> **note:** See Also: 

  * [*Oracle Database SQL Tuning Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=TGSQL-GUID-245F23B2-24AF-44D8-9F12-99FD1215E878) for information about configuring automatic optimizer statistics collection 



The SQL examples included previously are based on the following:
```
    DROP TABLE galaxies PURGE;
    CREATE TABLE galaxies (id NUMBER, name VARCHAR2(50), doc VARCHAR2(500), EMBEDDING VECTOR);
    -- or CREATE TABLE galaxies (id NUMBER, name VARCHAR2(50), doc VARCHAR2(500), EMBEDDING VECTOR(5,INT8));
    
    INSERT INTO galaxies VALUES (1, 'M31', 'Messier 31 is a barred spiral galaxy in the Andromeda constellation which has a lot of barred spiral galaxies.', '[0,2,2,0,0]');
    INSERT INTO galaxies VALUES (2, 'M33', 'Messier 33 is a spiral galaxy in the Triangulum constellation.', '[0,0,1,0,0]');
    INSERT INTO galaxies VALUES (3, 'M58', 'Messier 58 is an intermediate barred spiral galaxy in the Virgo constellation.', '[1,1,1,0,0]');
    INSERT INTO galaxies VALUES (4, 'M63', 'Messier 63 is a spiral galaxy in the Canes Venatici constellation.', '[0,0,1,0,0]');
    INSERT INTO galaxies VALUES (5, 'M77', 'Messier 77 is a barred spiral galaxy in the Cetus constellation.', '[0,1,1,0,0]');
    INSERT INTO galaxies VALUES (6, 'M91', 'Messier 91 is a barred spiral galaxy in the Coma Berenices constellation.', '[0,1,1,0,0]');
    INSERT INTO galaxies VALUES (7, 'M49', 'Messier 49 is a giant elliptical galaxy in the Virgo constellation.', '[0,0,0,1,1]');
    INSERT INTO galaxies VALUES (8, 'M60', 'Messier 60 is an elliptical galaxy in the Virgo constellation.', '[0,0,0,0,1]');
    INSERT INTO galaxies VALUES (9, 'NGC1073', 'NGC 1073 is a barred spiral galaxy in Cetus constellation.', '[0,1,1,0,0]');
```
    

For HNSW indexes:
```
    CREATE VECTOR INDEX galaxies_hnsw_idx ON galaxies (embedding)
    ORGANIZATION INMEMORY NEIGHBOR GRAPH
    DISTANCE COSINE
    WITH TARGET ACCURACY 95;
```
    

For IVF indexes:
```
    CREATE VECTOR INDEX galaxies_ivf_idx ON galaxies(embedding)
    ORGANIZATION NEIGHBOR PARTITIONS
    DISTANCE COSINE
    WITH TARGET ACCURACY 95;
```
    

**Parent topic:** [Optimizer Plans for Vector Indexes](optimizer-plans-vector-indexes.md)
