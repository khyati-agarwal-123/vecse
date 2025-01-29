## Querying an Inline External Table {#GUID-9D1D2768-77D9-49CC-8FC7-8474E1C6E294}

In this example, vectors in an external table of type `ORACLE_BIGDATA` are queried as part of a vector search. 
```
    select * from external (
    (
    COL1 vector,
    COL2 vector,
    COL3 vector,
    COL4 vector
    )
    TYPE ORACLE_BIGDATA
    DEFAULT DIRECTORY DEF_DIR1
    ACCESS PARAMETERS
    (
    com.oracle.bigdata.credential.name\=OCI_CRED
    com.oracle.bigdata.credential.schema\=PDB_ADMIN
    com.oracle.bigdata.fileformat=parquet
    com.oracle.bigdata.debug=true
    )
    location ( 'https://swiftobjectstorage.us-phoenix-1.oraclecloud.com/v1/myvdataproject/BIGDATA_PARQUET/vector_data/basic_vector_data.parquet' )
    REJECT LIMIT UNLIMITED
    ) tkexobd_bd_vector_inline;
```
    

**Parent topic:** [Vectors in External Tables](vectors-external-tables.md)
