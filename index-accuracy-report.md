## Index Accuracy Report {#GUID-A084929C-7A04-4764-9E5B-1204E0844CAF}

The index accuracy reporting feature lets you determine the accuracy of your vector indexes.

Accuracy of One Specific Query Vector

After a vector index is created, you may be interested to know how accurate your vector searches are. One possibility might be to run two queries using the same query vector, that is, one performing an approximate search using a vector index and the other performing an exact search without an index. Then, you need to manually compare the results to determine the real accuracy of your index.

Instead, you can use an index accuracy report provided by the `DBMS_VECTOR.INDEX_ACCURACY_QUERY` procedure. This procedure provides an accuracy report for a top-K index search for a specific query vector and a specific target accuracy. For syntax and parameter details, see [INDEX_ACCURACY_QUERY](index_accuracy_query.md#GUID-4DC6EDAD-659E-4093-B1CE-D116E0EC6A1D). 

Here is a usage example of this procedure using our galaxies scenario: 
```
    declare
    q_v VECTOR;
    report varchar2(128);
    begin
    q_v := to_vector('[0,1,1,0,0]');
    report := dbms_vector.index_accuracy_query(
    OWNER_NAME => 'COSMOS',
    INDEX_NAME => 'GALAXIES_HNSW_IDX',
    qv => q_v, top_K =>10,
    target_accuracy =>90 );
    dbms_output.put_line(report);
    end;
    /
```
    

The preceding example computes the top-10 accuracy of the `GALAXIES_HNSW_IDX` vector index using the embedding corresponding to the NGC 1073 galaxy and a 90% accuracy requested. 

The index accuracy report for this may look like: 
```
    Accuracy achieved (100%) is 10% higher than the Target Accuracy requested (90%)
```
    

The possible parameters are:

  * `owner_name`: Index owner name 

  * `index_name`: Index name 

  * `qv`: Query vector 

  * `top_K`: Top K value for accuracy computation 

  * `target_accuracy`: Target accuracy for the index 




Accuracy of Automatically Captured Query Vectors

An overloaded version of the `DBMS_VECTOR.INDEX_ACCURACY_REPORT` function allows you to capture from your past workloads, accuracy values achieved by your approximate searches using a particular vector index for a certain period of time. Query vectors used for approximate searches are captured automatically in memory and persisted to a catalog table every hour. 

The `INDEX_ACCURACY_REPORT` function computes the achieved accuracy using the captured query vectors for a given index. To compute the achieved accuracy for each query vector, the function compares the result set of approximate similarity searches with exact similarity searches for the same query vectors. 

The accuracy findings are stored in dictionary and exposed using the `DBA_VECTOR_INDEX_ACCURACY_REPORT` dictionary view. 

Here is a usage example of this function using the galaxies scenario:
```
    VARIABLE t_id NUMBER;
    BEGIN
    :t_id := DBMS_VECTOR.INDEX_ACCURACY_REPORT('VECTOR', 'GALAXIES_HNSW_IDX');
    END;
    /
```
    

You can also run the following statement to get the corresponding task identifier:
```
    SELECT DBMS_VECTOR.INDEX_ACCURACY_REPORT('VECTOR', 'GALAXIES_HNSW_IDX');
```
    

The following are possible parameters for the `INDEX_ACCURACY_REPORT` function: 

  * `owner_name` (IN): Index owner name 
  * `ind_name` (IN): Index name 
  * `start_time` (IN): Query vectors captured from this time are considered for the accuracy computation. A `NULL` `start_time` uses query vectors captured in the last 24 hours. 
  * `end_time` (IN): Query vectors captured until this time are considered for accuracy computation. A `NULL` `end_time` uses query vectors captured from `start_time` until the current time. 
  * Return Values: A numeric task ID if the accuracy for the given index was successfully computed. Otherwise, a `NULL` task ID is returned. 



> **note:** 

  * If both `start_time` and `end_time` are `NULL`, accuracy is computed using query vectors captured in the last 24 hours. 
  * If `start_time` is `NULL` and `end_time` is not `NULL`, accuracy is computed using query vectors captured between 24 hours before `end_time` until `end_time`. 
  * If `start_time` is not `NULL` and `end_time` is `NULL`, accuracy is computed using query vectors captured between `start_time` and the current time. 



You can see the analysis results using the `DBA_VECTOR_INDEX_ACCURACY_REPORT` view: 
```
    desc DBA_VECTOR_INDEX_ACCURACY_REPORT
```
```
    Name                                      Null?    Type
    ----------------------------------------- -------- ----------------------------
    TASK_ID                                            NUMBER
    TASK_TIME                                          TIMESTAMP(6)
    OWNER_NAME                                         VARCHAR2(128)
    INDEX_NAME                                         VARCHAR2(128)
    INDEX_TYPE                                         VARCHAR2(16)
    MIN_TARGET_ACCURACY                                NUMBER
    MAX_TARGET_ACCURACY                                NUMBER
    NUM_VECTORS                                        NUMBER
    MEDIAN_ACHIEVED_ACCURACY                           NUMBER
    MIN_ACHIEVED_ACCURACY                              NUMBER
    MAX_ACHIEVED_ACCURACY                              NUMBER
```
    

Select target accuracy values in the following statement:
```
    SELECT MIN_TARGET_ACCURACY, MAX_TARGET_ACCURACY, num_vectors, MIN_ACHIEVED_ACCURACY, MEDIAN_ACHIEVED_ACCURACY, MAX_ACHIEVED_ACCURACY
    FROM DBA_VECTOR_INDEX_ACCURACY_REPORT WHERE task_id = 1;
```
```
    MIN_TARGET_ACCURACY MAX_TARGET_ACCURACY NUM_VECTORS MIN_ACHIEVED_ACCURACY MEDIAN_ACHIEVED_ACCURACY MAX_ACHIEVED_ACCURACY
    ------------------- ------------------- ----------- --------------------- ------------------------ ---------------------
    1                  10           2                    49                       57                    65
    11                  20           3                    60                       73                    83
    21                  30           3                    44                       64                    84
    31                  40           2                    63                     76.5                    90
    41                  50           3                    63                       81                    90
    61                  70           2                    57                       68                    79
    71                  80           3                    79                       87                    89
    81                  90           3                    70                       71                    78
    91                 100           4                    67                     79.5                    88
```
    

Each row in the output represents a bucket of 10 target accuracy values: 1-10, 11-20, 21-30, … , 91-100.

Consider the following partial statement that runs an approximate similarity search for a particular query vector and a particular target accuracy:
```
    SELECT ...
    FROM ...
    WHERE ...
    ORDER BY VECTOR_DISTANCE( embedding, :my_query_vector, COSINE )
    FETCH APPROXIMATE FIRST 3 ROWS ONLY WITH TARGET ACCURACY 65;
```
    

Once the preceding approximate similarity search is run and captured by your accuracy report task, the value of `NUM_VECTORS` would be increased by one in row 6 (bucket values between 61 and 70) of the results of the select statement on the `DBA_VECTOR_INDEX_ACCURACY_REPORT` view for your task. `NUM_VECTORS` represents the number of query vectors that fall in a particular target accuracy bucket. 

`MIN_ACHIEVED_ACCURACY`, `MEDIAN_ACHIEVED_ACCURACY`, and `MAX_ACHIEVED_ACCURACY` are the actual achieved accuracy values for the given target accuracy bucket. 

> **note:** The initialization parameter `VECTOR_QUERY_CAPTURE` is used to enable and disable capture of query vectors. The parameter value is set to `ON` by default. You can turn off this background functionality by setting `VECTOR_QUERY_CAPTURE` to `OFF`. When `VECTOR_QUERY_CAPTURE` is `ON`, the database captures some of the query vectors through sampling. The captured query vectors are retrained for a week and then purged automatically. 

**Parent topic:** [Manage the Different Categories of Vector Indexes](manage-different-categories-vector-indexes.md)
