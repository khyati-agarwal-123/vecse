## Included Columns {#GUID-641E6D17-6B59-4860-A5BD-DF3D33793D43}

Included columns in vector indexes facilitate faster searches with attribute filters by incorporating non-vector columns within a Neighbor Partition Vector Index.

Consider a classic example where you are analyzing the home price data. You want only the top 10 homes from your description where the price is less than 2 million dollars and/or from the specific zip code. Or to a finer granularity, you just need to retrieve only the top home that matches the description and your filter attribute. Included columns make this possible by incorporating filterable attributes, such as price and zip code, directly into the vector index. This integration allows the index to evaluate and return results based on both the vector similarity and the attribute filters, without requiring additional steps to cross-reference the base table. 

By bridging the gap between traditional database filters and AI-powered vector searches, included columns deliver faster, more efficient results. This synergy enhances search performance, especially in use cases requiring fine-grained filtering alongside advanced similarity computations.

Syntax and Specifications

Create a base table, which contains the sales data for all products sold in the current year: 
```
    create table houses(id number, zip_code number, price number,
    description clob, data_vector vector);
```
    

Then, you can create an IVF index **with included column on price** as follows: 
```
    create vector index vidx_ivf on houses(data_vector) **include (price)**
    organization neighbor partitions with target accuracy 95
    distance EUCLIDEAN parameters(type IVF, neighbor partitions 2);
```
    

The above syntax creates a vector index `VIDX_IVF` with `organization` as `NEIGHBOR PARTITIONS` on the `DATA_VECTOR` column where the `PRICE` column is an included column. The `INCLUDE` keyword allows a user to specify the attributes they wish to include in the IVF index. 

> **note:** Included columns does not work with partition local indexes. 

Recall that the IVF index has two auxiliary tables : Centroids table - `IVF_FLAT_CENTROIDS` and the Centroid Partitions table - `IVF_FLAT_CENTROID_PARTITIONS`. The following code block describes the Centroid Partitions table without the included columns: 
```
    Name					   Null?    Type
    ----------------------------------------- -------- ----------------------------
    BASE_TABLE_ROWID			   NOT NULL ROWID
    CENTROID_ID				   NOT NULL NUMBER
    DATA_VECTOR					    VECTOR(384, FLOAT32, DENSE)
    --------------------------------------------------------------------------------
```
    

The list of included columns are augmented in the `IVF_FLAT_CENTROID_PARTITIONS` table of IVF index. The BASE_TABLE_ROWID column can be thought of as an implicit included column. The following table describes the `IVF_FLAT_CENTROID_PARTITIONS` table with the included column augmented as the last row. In this case, the `price`.
```
    Name					   Null?    Type
    ----------------------------------------- -------- ----------------------------
    BASE_TABLE_ROWID			   NOT NULL ROWID
    CENTROID_ID				   NOT NULL NUMBER
    DATA_VECTOR					    VECTOR(384, FLOAT32, DENSE)
    **PRICE						    NUMBER**
    --------------------------------------------------------------------------------
```
    

DML operations on the base table also maintain the included columns. Updating any row on the base table for the included columns will all also update the centroid partitions for the IVF index. 

> **note:** **Restrictions on INCLUDE :**

The `INCLUDE` list can contain:​ 

  * Limited to a maximum of 31 columns - Oracle indexes support 32 key columns. 
  * Supports only the following data types : `NUMBER`, `CHAR`, `VARCHAR2`, `DATE`, `TIMESTAMP`, `RAW`, and `JSON` (limited). 
  * JSON columns are limited to a maximum of 32k and this needs to be specified in the DDL.. So, when you are creating a table, you have to explicitly include the `limit` field. A sample example is shown below:
```
    CREATE TABLE json_vector_data(id NUMBER,json_data JSON (**limit** 32000),vector_data vector);
```
    

  * Does not support NLS types, reference-based LOB types (`BLOB`, `CLOB` etc.,) or `LONG` types. 



Benefits of Using Included Columns

  1. **IVF No-Filter with and without Included Columns**: 

If you consider a non-filter query that fetches the two homes nearest to a specified vector, the resulting query will result in an execution plan with multiple joins.
```
    select /*+ VECTOR_INDEX_TRANSFORM(t) */ price from houses t order by
    VECTOR_DISTANCE(data_vector, :query_vector) fetch first 2 rows only;
```
    

> **note:** `query_vector` contains the actual input vector. You can use the instructions mentioned in [SQL Quick Start Using a FLOAT32 Vector Generator](sql-quick-start-using-float32-vector-generator.md#GUID-1AA9AA2B-F3A6-481B-9EE3-8B08A4CE200D) to generate query_vector. 

The multiple joins are displayed in the execution plan below: 
```
    Execution Plan
    ----------------------------------------------------------
    Plan hash value: 466477707
    
    ------------------------------------------------------------------------------------------------------
    | Id  | Operation			| Name							     |
    ------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT		|							     |
    |   1 |  VIEW				|							     |
    |   2 |   NESTED LOOPS			|							     |
    |   3 |    VIEW 			| VW_IVPSR_11E7D7DE					     |
    |*  4 |     COUNT STOPKEY		|							     |
    |   5 |      VIEW			| VW_IVPSJ_578B79F1					     |
    |*  6 |       SORT ORDER BY STOPKEY	|							     |
    |   7 |        NESTED LOOPS		|							     |
    |   8 | 	VIEW			| VW_IVCR_B5B87E67					     |
    |*  9 | 	 COUNT STOPKEY		|							     |
    |  10 | 	  VIEW			| VW_IVCN_9A1D2119					     |
    |* 11 | 	   SORT ORDER BY STOPKEY|							     |
    |  12 | 	    TABLE ACCESS FULL	| VECTOR$VIDX_IVF$74478_74488_0$IVF_FLAT_CENTROIDS	     |
    |  13 | 	PARTITION LIST ITERATOR |							     |
    |* 14 | 	 TABLE ACCESS FULL	| VECTOR$VIDX_IVF$74478_74488_0$IVF_FLAT_CENTROID_PARTITIONS |
    |  15 |    TABLE ACCESS BY USER ROWID	| HOUSES						     |
    ------------------------------------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
    4 - filter(ROWNUM<=2)
    6 - filter(ROWNUM<=2)
    9 - filter(ROWNUM<=1)
    11 - filter(ROWNUM<=1)
    14 - filter("VW_IVCR_B5B87E67"."CENTROID_ID"="VTIX_CNPART"."CENTROID_ID")
```
    

You can see in the execution plan that the IVF centroids are joined with the centroid partition table to scan for the centroids in the nearest neighbor partitions fetched from the centroid table. This is operation is expected. However you are also performing a join with the base table: HOUSES to apply the attribute filters. This second join against the base table could be avoided if the columns were included in the centroid partitions table.

The following execution plan displays the same operation when using a neighbor partition vector index (IVF) created with included columns:
```
    Execution Plan
    ----------------------------------------------------------
    Plan hash value: 504484986
    
    -------------------------------------------------------------------------------------------------------
    | Id  | Operation		       | Name							      |
    -------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT	       |							      |
    |   1 |  VIEW			       |							      |
    |   2 |   VIEW			       | VW_IVPSR_11E7D7DE					      |
    |*  3 |    COUNT STOPKEY	       |							      |
    |   4 |     VIEW		       | VW_IVPSJ_578B79F1					      |
    |*  5 |      SORT ORDER BY STOPKEY     |							      |
    |   6 |       NESTED LOOPS	       |							      |
    |   7 |        VIEW		       | VW_IVCR_B5B87E67					      |
    |*  8 | 	COUNT STOPKEY	       |							      |
    |   9 | 	 VIEW		       | VW_IVCN_9A1D2119					      |
    |* 10 | 	  SORT ORDER BY STOPKEY|							      |
    |  11 | 	   TABLE ACCESS FULL   | VECTOR$VIDX_IVF_1$74483_74508_0$IVF_FLAT_CENTROIDS	      |
    |  12 |        PARTITION LIST ITERATOR |							      |
    |* 13 | 	TABLE ACCESS FULL      | VECTOR$VIDX_IVF_1$74483_74508_0$IVF_FLAT_CENTROID_PARTITIONS |
    -------------------------------------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
    3 - filter(ROWNUM<=2)
    5 - filter(ROWNUM<=2)
    8 - filter(ROWNUM<=1)
    10 - filter(ROWNUM<=1)
    13 - filter("VW_IVCR_B5B87E67"."CENTROID_ID"="VTIX_CNPART"."CENTROID_ID")
```
    

As you can see in the execution plan above, adding included columns to a vector index ensures there is no base table pre-filter evaluation required. The query can be resolved by scanning the centroids partitions table for the neighbor partitioned vector index.

  2. **IVF Filtering with Included Columns**: 

One of the more commonly used plans for neighbor graph vector indexes is the pre-filter plan. In this case you are filtering the contents of the base table to eliminate non-relevant rows. But evaluating a query without using a filter attribute can be very expensive as the operation involves several joins. The following query uses a neighbor partition vector index without included columns:
```
    SELECT /*+VECTOR_INDEX_TRANSFORM(houses, vidx_ivf) */ price FROM houses
    WHERE price = 1400000 ORDER BY vector_distance(data_vector, :query_vector)
    FETCH APPROX FIRST 4 ROWS ONLY;
```
    

> **note:** `query_vector` contains the actual input vector. You can use the instructions mentioned in [SQL Quick Start Using a FLOAT32 Vector Generator](sql-quick-start-using-float32-vector-generator.md#GUID-1AA9AA2B-F3A6-481B-9EE3-8B08A4CE200D) to generate query_vector. 

The execution plan shows multiple joins and the mandatory join with the base table `HOUSES`. 
```
    Execution Plan
    ----------------------------------------------------------
    Plan hash value: 3359903466
    
    -------------------------------------------------------------------------------------------------------------
    | Id  | Operation			       | Name							    |
    -------------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT		       |							    |
    |*  1 |  COUNT STOPKEY			       |							    |
    |   2 |   VIEW				       |							    |
    |*  3 |    SORT ORDER BY STOPKEY	       |							    |
    |   4 |     NESTED LOOPS		       |							    |
    |   5 |      MERGE JOIN CARTESIAN	       |							    |
    |*  6 |       TABLE ACCESS FULL 	       | HOUSES 						    |
    |   7 |       BUFFER SORT		       |							    |
    |   8 |        VIEW			       | VW_IVCR_2D77159E					    |
    |*  9 | 	COUNT STOPKEY		       |							    |
    |  10 | 	 VIEW			       | VW_IVCN_9A1D2119					    |
    |* 11 | 	  SORT ORDER BY STOPKEY        |							    |
    |  12 | 	   TABLE ACCESS FULL	       | VECTOR$VIDX_IVF$74478_74488_0$IVF_FLAT_CENTROIDS	    |
    |* 13 |      TABLE ACCESS BY GLOBAL INDEX ROWID| VECTOR$VIDX_IVF$74478_74488_0$IVF_FLAT_CENTROID_PARTITIONS |
    |* 14 |       INDEX UNIQUE SCAN 	       | SYS_C008791						    |
    -------------------------------------------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
    1 - filter(ROWNUM<=4)
    3 - filter(ROWNUM<=4)
    6 - filter("HOUSES"."PRICE"=1400000)
    9 - filter(ROWNUM<=3)
    11 - filter(ROWNUM<=3)
    13 - filter("VW_IVCR_2D77159E"."CENTROID_ID"="VTIX_CNPART"."CENTROID_ID")
    14 - access("HOUSES".ROWID="VTIX_CNPART"."BASE_TABLE_ROWID")
```
    

On the other hand, if you use the same in-filter query on the IVF index created with `price` as the `included` column, the execution plan would show less joins, the join with the base table is avoided. This is shown in the plan below:
```
    Execution Plan
    ----------------------------------------------------------
    Plan hash value: 4183240211
    
    -------------------------------------------------------------------------------------------------------
    | Id  | Operation		       | Name							      |
    -------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT	       |							      |
    |*  1 |  COUNT STOPKEY		       |							      |
    |   2 |   VIEW			       |							      |
    |*  3 |    SORT ORDER BY STOPKEY       |							      |
    |*  4 |     HASH JOIN		       |							      |
    |   5 |      PART JOIN FILTER CREATE   | :BF0000						      |
    |   6 |       VIEW		       | VW_IVCR_2D77159E					      |
    |*  7 |        COUNT STOPKEY	       |							      |
    |   8 | 	VIEW		       | VW_IVCN_9A1D2119					      |
    |*  9 | 	 SORT ORDER BY STOPKEY |							      |
    |  10 | 	  TABLE ACCESS FULL    | VECTOR$VIDX_IVF_1$74483_74508_0$IVF_FLAT_CENTROIDS	      |
    |  11 |      PARTITION LIST JOIN-FILTER|							      |
    |* 12 |       TABLE ACCESS FULL        | VECTOR$VIDX_IVF_1$74483_74508_0$IVF_FLAT_CENTROID_PARTITIONS |
    -------------------------------------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
    1 - filter(ROWNUM<=4)
    3 - filter(ROWNUM<=4)
    4 - access("VW_IVCR_2D77159E"."CENTROID_ID"="VTIX_CNPART"."CENTROID_ID")
    7 - filter(ROWNUM<=3)
    9 - filter(ROWNUM<=3)
    12 - filter("VTIX_CNPART"."PRICE"=1400000)
```
    




**Parent topic:** [Neighbor Partition Vector Index](neighbor-partition-vector-index.md)
