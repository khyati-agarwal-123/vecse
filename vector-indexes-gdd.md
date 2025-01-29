## Vector Indexes in a Globally Distributed Database {#GUID-7FF5081C-B48F-4B87-A489-D408D8CEA34A}

Inverted File Flat (IVF) index, Hierarchical Navigable Small World (HNSW) index, and Hybrid Vector Index (HVI) are supported on sharded tables in a distributed database; however there are some considerations.

> **note:** 

Global indexes are not supported on sharded tables; however, this limitation does not exist for the global HNSW and IVF index.

Inverted File Flat Index

Inverted File Flat Index (IVF Flat or simply IVF) is a partitioned-based index that lets you balance high-search quality with reasonable speed.

You can create a local IVF index on vector columns in a sharded table. There is no syntax change required. However, you must create the index partitions, partitions of `$IVF_FLAT_CENTROIDS` and `$IVF_FLAT_CENTROID_PARTITIONS` in relevant chunk tablespaces to facilitate move chunks across shards. 

  * IVF indexes and HNSW indexes on a sharded table must be created on the shard catalog database with `SHARD DDL` enabled. 

  * The `CREATE INDEX` command is propagated as is to all of the shards by the shard coordinator. The `CREATE INDEX` clause scope is the shard. 




There is no syntax change to create an IVF index on a sharded table, when compared to the syntax to create an IVF index on a non-sharded table.
```
    CREATE VECTOR INDEX ivf_image
    ON houses (image)
    ORGANIZATION NEIGHBOR PARTITIONS WITH TARGET ACCURACY 95
    DISTANCE EUCLIDEAN PARAMETERS
    (type IVF, NEIGHBOR PARTITIONS 1000) PARALLEL 16;
```
    

Hierarchical Navigable Small World Index

There is no syntax change to create a Hierarchical Navigable Small World (HNSW) index on a sharded table, when compared to the syntax to create an HNSW index on a non-sharded table.
```
    CREATE VECTOR INDEX hnsw_image
    ON houses (image)
    ORGANIZATION INMEMORY NEIGHBOR GRAPH
    WITH TARGET ACCURACY 95;
```
    

Hybrid Vector Index

The Hybrid Vector Index (HVI) is an index that allows you to easily index and query documents using a combination of full text search and semantic vector search to achieve higher quality search results. 

Creation and use of HVI is supported across shards. For example:
```
    CREATE HYBRID VECTOR INDEX IDX ON T1(TEXT)
    PARAMETERS('MODEL HVI_TEST.DATABASE VECTOR_IDXTYPE IVF');
```
    

The steps to create an HVI in a distributed database is similar to creating a HVI on a non-sharded table, except the step to load the model on the catalog and shards.

To have models ready for loading, first copy the zipped model file into the working directory. Once copied, unzip the model file and ensure that privileges are granted to all-shard users.
```
    alter session enable shard ddl;
    create user shdusr identified by shdusr;
    grant all privileges to shdusr;
    grant dba to shdusr;
    grant connect, ctxapp, unlimited tablespace, create table,
    create procedure to shdusr;
    **grant create mining model to shdusr; **
    create or replace directory CTX_WORK_DIR as <$T_WORK>;
    grant read on directory CTX_WORK_DIR to shdusr;
```
    

Loading the model needs to be performed on the catalog as well as all of the shards. This ensures that the catalog and shards have the infrastructure to create an HVI. 

To ensure uniformity, execute the `dbms_vector.load_onnx_model` procedure in a single step. Because this is a PL/SQL procedure, use the `exec_shard_plsql` procedure to run it on all of the instances. `Exec_shard_plsql` is a sharding-specific procedure that allows you to execute PL/SQL procedures on catalog and shards as if executing a sharding DDL. 

Run the following command on the catalog, which will load the model on the catalog and on all shards:
```
    EXEC sys.exec_shard_plsql('sys.dbms_vector.load_onnx_model(''CTX_WOR
    K_DIR'', ''tmnx046.onnx'', ''database'', json(''{"function":
    "embedding", "embeddingOutput": "embedding", "input": {"input":
    ["DATA"]}}''))', 5);
```
    

When querying using the HVI there is no syntax change required.

**Parent topic:** [Create Vector Indexes and Hybrid Vector Indexes](create-vector-indexes-and-hybrid-vector-indexes.md)
