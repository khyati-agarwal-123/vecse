## Inverted File Flat Index Syntax and Parameters {#GUID-FC314C40-1018-46B9-9F1C-660BBE28FBE9}

Review the syntax and examples for Inverted File Flat vector indexes.

Syntax
```
    CREATE VECTOR INDEX
    ON  (  )
    [GLOBAL] ORGANIZATION [NEIGHBOR] PARTITIONS
    [WITH] [DISTANCE ]
    [WITH TARGET ACCURACY
    [PARAMETERS ( TYPE IVF, { NEIGHBOR PARTITIONS  \|
    SAMPLES_PER_PARTITION  \|
    MIN_VECTORS_PER_PARTITION  }
    )]]
    [PARALLEL ]
    [LOCAL];
```
    

The `GLOBAL` clause specifies a global IVF index. By default, IVF vector indexes are globally partitioned by centroid. 

Specify the `LOCAL` clause to create a local IVF index. 

IVF PARAMETERS

`NEIGHBOR PARTITIONS` determines the target number of centroid partitions that are created by the index. 

`SAMPLES_PER_PARTITION` decides the total number of vectors that are passed to the clustering algorithm (`samples_per_partition * neighbor_partitions`). Passing all the vectors could significantly increase the total index creation time. The goal is to pass in a subset of vectors that can capture the data distribution. 

`MIN_VECTORS_PER_PARTITION` represents the target minimum number of vectors per partition. The goal is to trim out any partition that can end up with few vectors (<= 100). 

The valid range for IVF vector index parameters are:

  * `ACCURACY`: > 0 and <= 100 
  * `DISTANCE`: `EUCLIDEAN`, `L2_SQUARED` (aka `EUCLIDEAN_SQUARED`), `COSINE`, `DOT`, `MANHATTAN`, `HAMMING`
  * `TYPE`: `IVF`
  * `NEIGHBOR PARTITIONS`: >0 and <= 10000000 
  * `SAMPLES_PER_PARTITION`: from 1 to (num_vectors/neighbor_partitions) 
  * `MIN_VECTORS_PER_PARTITION`: from 0 (no trimming of centroid partitions) to total number of vectors (would result in 1 centroid partition) 



Examples

  * To create a global IVF index:
```
    CREATE VECTOR INDEX galaxies_ivf_idx ON galaxies (embedding)
    ORGANIZATION NEIGHBOR PARTITIONS
    DISTANCE COSINE
    WITH TARGET ACCURACY 95;
```
```
    CREATE VECTOR INDEX galaxies_ivf_idx ON galaxies (embedding)
    ORGANIZATION NEIGHBOR PARTITIONS
    DISTANCE COSINE
    WITH TARGET ACCURACY 90 PARAMETERS (type IVF, neighbor partitions 10);
```
    

  * To create a local IVF index:
```
    CREATE VECTOR INDEX galaxies_ivf_idx ON galaxies (embedding)
    ORGANIZATION NEIGHBOR PARTITIONS
    DISTANCE COSINE
    WITH TARGET ACCURACY 90 PARAMETERS (type IVF, neighbor partitions 10)
    LOCAL;
```
    




For detailed information on the syntax, see [CREATE VECTOR INDEX](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SQLRF-GUID-B396C369-54BB-4098-A0DD-7C54B3A0D66F) in *Oracle Database SQL Language Reference*. 

**Related Topics**

  * [Inverted File Flat Vector Indexes Partitioning Schemes](inverted-file-flat-vector-indexes-partitioning-schemes.md#GUID-98797C70-269E-42E9-B24B-1E3C461D5932)



**Parent topic:** [Neighbor Partition Vector Index](neighbor-partition-vector-index.md)
