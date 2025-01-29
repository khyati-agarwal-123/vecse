## Guidelines and Restrictions for Hybrid Vector Indexes {#GUID-8CFB2031-695D-4C9F-92AF-6628124DE82A}

Review these guidelines and the current set of restrictions when working with hybrid vector indexes.

Index Creation Guidelines

A hybrid vector index is a single domain index that includes both Oracle Text search index and Oracle AI Vector Search vector index. Any failure during the construction of one index type can cause the entire index creation to fail as a whole. Therefore, you must adhere to all the indexing guidelines that are individually applicable to a vector index and an Oracle Text search index.

For example, if there is insufficient vector memory pool for the HNSW vector index or if there is inadequate tablespace, then the vector index-related error can lead to a failure in creating a hybrid vector index.

  * If a destructive operation occurs, such as running `Ctrl+C` or shutting down the database, then you must check the status of the index. If the index status does not display as `valid`, it indicates that the index may be corrupted or inconsistent, making it unsafe for use. In such cases, you must recreate the index. 

  * If there are any errors in the hybrid vector index DDL or if there is any DDL failure, then you must manually drop the index and create it again. Before using the index, ensure that it is safe to use by checking the index status.




To check the status of your index, you can query the regular Oracle Text data dictionary views, such as `CTX_INDEXES` and `CTX_USER_INDEXES`. See [*Oracle Text Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-A3660501-A4A9-4EEE-9811-4229E93F7A9C). 

Hybrid Vector Index Maintenance Operations

You can maintain your hybrid vector indexes just like an Oracle Text index. A hybrid vector index supports mostly all the traditional operations of Oracle Text indexes, such as Synchronization, Optimization, and Automatic Maintenance.

Here are some guidelines on maintaining indexes after DML operations on base tables:

  * **Default index maintenance**: 

A hybrid vector index runs in an automatic maintenance mode (`MAINTENANCE AUTO`), which means that your DMLs are automatically synchronized into the index in the background at optimal intervals. 

You do not need to manually configure a maintenance type or synchronization type for maintaining a hybrid vector index. However, if required, you can modify the default settings to set any synchronization interval, specify a `SYNC` type (such as `MANUAL`, `EVERY`, or `ON COMMIT`), or schedule any background synchronization job using the `DBMS_SCHEDULER`. 

In an automatic maintenance mode, indexes are asynchronously maintained without any user intervention. Oracle recommends that you periodically examine regular Oracle Text views to know the status of all background maintenance events. 

For information on these views, see [*Oracle Text Application Developer's Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCAPP-GUID-440A629A-F9A6-4303-84BD-02429CDA7D05). 

  * **Default index optimization**: 

A hybrid vector index runs with an automatic background Optimize Full job every midnight "local" time. This job optimizes your index to defragment it and clean up any lazy deletes from secondary tables such as the `$I`, `$D`, and `$VR`. 

  * **Optimizing the $VR Table**

For a hybrid vector index, you can also optimize only the `$VR` table. This optimization is run in “section mode” for the new “section” (semantic index). This will compact the `$VR` table only by removing any deleted ROWIDs. 
```
    CTX_DDL.OPTIMIZE_INDEX([schema.]index_name, ‘token_type’,
    ‘ctx_ddl.section_semantic_index’);
```
    

Oracle recommends you to explicitly run index optimization more frequently, especially if your hybrid vector index involves a large number of inserts, updates, or deletes to base tables. This is because frequent index synchronization can cause fragmentation of your index, which can adversely affect query response time. You can reduce fragmentation and index size by optimizing the index with `CTX_DDL.OPTIMIZE_INDEX`. 

For information on how to configure optimization methods, see the index maintenance preferences in [CREATE HYBRID VECTOR INDEX](create-hybrid-vector-index.md#GUID-22B07106-419E-4C55-B4E1-3FD691E033BC). 




Guidelines to Run Direct SQL Queries on $VR

If you query secondary tables such as `$VR` directly before an index optimization job is run, you may still see old data or deleted rows because these are ignored at query time by Oracle provided search API (`DBMS_HYBRID_VECTOR.SEARCH`). For example, you may see lazy deletes that mark the row IDs as deleted but are only removed from secondary tables after the index is optimized. 

Instead of examining the `$VR` table directly, you can use the data dictionary view `*``*$VECTORS` to work with the contents of the `$VR` table, and accordingly run direct SQL queries on the view. This view lets you query the document table's row IDs by excluding all lazy deletes (along with corresponding chunks and embeddings that are generated). 

Note that the `*``*$VECTORS` view resides in a user's schema. Therefore, if a hybrid vector index is named `IDX`, then the view name is `IDX$VECTORS`. 

For information on this view, see [$VECTORS](indexnamevectors.md#GUID-2B53FFD8-D2A4-469D-83B1-C653602FFAEE). 

Guidelines for VPD Policy Protection

If you have applied an Oracle Virtual Private Database (VPD) policy to the base table, then you must:

  * Protect the entire hybrid vector index with the same VPD policy, including the `$I`, `$VR`, and `$D` secondary tables. 

  * Protect the `*``*$VECTORS` view (which is created on the `$VR` table) with the same VPD policy. 

  * Ensure that any SQL queries run on the hybrid vector index and any direct SQL queries run on secondary tables that are created within the schema also respect that VPD policy. 




For information on how to use VPD to control data access in general, see [*Oracle Database Security Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=DBSEG-GUID-06022729-9210-4895-BF04-6177713C65A7). 

Hybrid Vector Index DDL Restrictions

  * The supported data types are `CLOB`, `VARCHAR2`, or `BLOB`. Textual columns with the `IS JSON` check constraint are not supported. 

Other data types such as JSON are currently not supported for a hybrid vector index.

  * Creating a local hybrid vector index is not supported.

  * You currently cannot import externally created chunks or vectors directly into the `$VR` table of a hybrid vector index. 

  * Currently, only in-database ONNX embedding models are supported (not third-party embedding models) to generate vector embeddings for a hybrid vector index.

  * The `HAMMING` and `JACCARD` distance metrics are currently not supported with hybrid vector indexes. 

  * The DML tracking support aligns with the behavior of the IVF and HNSW vector indexes for the vector index part of a hybrid vector index.

  * Creating materialized views based on the outputs produced by the `VECTOR_CHUNKS`, `UTL_TO_CHUNKS`, `UTL_TO_TEXT`, and `UTL_TO_SUMMARY` functions is currently not supported. 




> **note:** 

With the current set of restrictions, Oracle recommends that you create a hybrid vector index in the following scenarios:

  * If you want to search through textual documents (with `CLOB`, `VARCHAR2`, or `BLOB` data types) in a hybrid search mode. 

  * If you want to alternatively switch between keyword search and semantic search when querying textual data.




If you have the data that will only be searched semantically or if you want to manually create your own third-party vector embeddings (using the `DBMS_VECTOR` and `DBMS_VECTOR_CHAIN` third-party REST APIs), then use a vector index instead of creating a hybrid vector index. 

**Related Topics**

  * [Understand Hybrid Vector Indexes](understand-hybrid-vector-indexes.md#GUID-28C18166-43BB-4D2C-B8B3-8127D3485578)



**Parent topic:** [Manage Hybrid Vector Indexes](manage-hybrid-vector-indexes.md)
