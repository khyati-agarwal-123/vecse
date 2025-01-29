## Performing a Semantic Similarity Search Using External Table {#GUID-09B1CA70-CE37-4D4D-B68A-E8BE27ED1759}

See a SQL example plan that illustrates how you can use external tables as the data set for semantic similarity searches

The following is an example of an explain plan for `select id`, embedding from `ext_table_3`, and using `order by vector_distance('[1,1]', embedding, cosine)` to return approximately only the first three rows of data with a target accuracy of 90 percent: 
```
    SQL> select * from table(dbms_xplan.display('plan_table', null, 'advanced predicate'));
    
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    Plan hash value: 1784440045
    
    --------------------------------------------------------------------------------
    ---------------------
    
    | Id  | Operation                     | Name        | Rows  | Bytes |TempSpc| Co
    st (%CPU)| Time     |
    
    --------------------------------------------------------------------------------
    ---------------------
    
    
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT              |             |     3 | 48945 |       |  1
    466K  (2)| 00:00:58 |
    
    |*  1 |  COUNT STOPKEY                |             |       |       |       |
    |          |
    
    |   2 |   VIEW                        |             |   102K|  1588M|       |  1
    466K  (2)| 00:00:58 |
    
    |*  3 |    SORT ORDER BY STOPKEY      |             |   102K|  1589M|   798M|  1
    466K  (2)| 00:00:58 |
    
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    
    |   4 |     EXTERNAL TABLE ACCESS FULL| EXT_TABLE_3 |   102K|  1589M|       |
    362   (7)| 00:00:01 |
    
    --------------------------------------------------------------------------------
    ---------------------
    
    
    Query Block Name / Object Alias (identified by operation id):
    -------------------------------------------------------------
    
    
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    1 - SEL$2
    2 - SEL$1 / "from$_subquery$_002"@"SEL$2"
    3 - SEL$1
    4 - SEL$1 / "EXT_TABLE_3"@"SEL$1"
    
    Outline Data
    -------------
    
    /*+
    BEGIN_OUTLINE_DATA
    FULL(@"SEL$1" "EXT_TABLE_3"@"SEL$1")
    
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    NO_ACCESS(@"SEL$2" "from$_subquery$_002"@"SEL$2")
    OUTLINE_LEAF(@"SEL$2")
    OUTLINE_LEAF(@"SEL$1")
    ALL_ROWS
    OPT_PARAM('_fix_control' '6670551:0')
    OPT_PARAM('_optimizer_cost_model' 'fixed')
    DB_VERSION('26.1.0')
    OPTIMIZER_FEATURES_ENABLE('26.1.0')
    IGNORE_OPTIM_EMBEDDED_HINTS
    END_OUTLINE_DATA
    */
    
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    
    Predicate Information (identified by operation id):
    ---------------------------------------------------
    
    1 - filter(ROWNUM<=3)
    3 - filter(ROWNUM<=3)
    
    Column Projection Information (identified by operation id):
    -----------------------------------------------------------
    
    1 - "from$_subquery$_002"."ID"[NUMBER,22], "from$_subquery$_002"."EMBEDDING"[
    
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    VECTOR,32600]
    
    2 - "from$_subquery$_002"."ID"[NUMBER,22], "from$_subquery$_002"."EMBEDDING"[
    VECTOR,32600]
    
    3 - (#keys=1) VECTOR_DISTANCE(VECTOR('[1,1]', *, *, * /*+  USEBLOBPCW_QVCGMD
    */ ),
    
    "EMBEDDING" /*+ LOB_BY_VALUE */ , COSINE)[BINARY_DOUBLE,8], "ID"[NUMBER,2
    2], "EMBEDDING" /*+
    
    
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    LOB_BY_VALUE */ [VECTOR,32600]
    4 - "ID"[NUMBER,22], "EMBEDDING" /*+ LOB_BY_VALUE */ [VECTOR,32600],
    VECTOR_DISTANCE(VECTOR('[1,1]', *, *, * /*+  USEBLOBPCW_QVCGMD */ ), "EMB
    EDDING" /*+
    
    LOB_BY_VALUE */ , COSINE)[BINARY_DOUBLE,8]
    
    Query Block Registry:
    ---------------------
    
    SEL$1 (PARSER) [FINAL]
    
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    SEL$2 (PARSER) [FINAL]
```
    

+

**Parent topic:** [Vectors in External Tables](vectors-external-tables.md)
