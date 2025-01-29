## Vectors in Distributed Database Tables {#GUID-F75FB9CA-E7D1-46BC-847A-7324EF21D829}

There is no new SQL syntax or keyword when creating sharded tables and duplicated tables with vector columns in a Globally Distributed Database; however, there are some requirements and restrictions to consider.

User Permissions

Only an all-shards user can create sharded and duplicated tables. You must connect to the shard catalog as an all-shards user. Connecting to the shard catalog as an all-shards user automatically enables `SHARD DDL`, and the DDL to create the tables is propagated to all the shards in the distributed database. 

Creating Sharded Tables with a Vector Column

  * Sharded tables must be created on the catalog database with `SHARD DDL` enabled. 

  * A vector column cannot be part of the sharding key or the partitionset key.

  * The `CREATE SHARDED TABLE` command is propagated to all of the shards by the shard coordinator. 




The syntax to create a sharded table with a vector column is same as the syntax to create a non-sharded table with a vector column. The only difference is to include the `SHARDED` keyword in the `CREATE TABLE` statement. 
```
    CREATE SHARDED TABLE REALTORS(
    REALTOR_ID NUMBER PRIMARY KEY,
    NAME VARCHAR2(20),
    IMAGE VECTOR,
    ZIPCODE VARCHAR2(40))
    PARTITION BY CONSISTENT HASH(REALTOR_ID)
    TABLESPACE SET TS1;
```
    

Creating Duplicated Tables with a Vector Column

  * Duplicated tables must be created on the shard catalog database with `SHARD DDL` enabled. 




The syntax to create a duplicated table with a vector column is same as the syntax to create a non-sharded table with a vector column. The only difference is to include the `DUPLICATED` keyword in the `CREATE TABLE` statement. 
```
    CREATE DUPLICATED TABLE PRODUCT_DESCRIPTIONS
    (
    PRODUCT_ID          NUMBER(6,0) NOT NULL,
    ORDER_ID            NUMBER(6,0) NOT NULL,
    LANGUAGE_ID         VARCHAR2(6 BYTE),
    TRANSLATED_NAME     NVARCHAR2(50),
    TRANSLATED_DESCRIPTION NVARCHAR2(2000),
    VECT4 VECTOR,
    VECT5 VECTOR,
    CONSTRAINT  PRODUCT_DESCRIPTIONS_PK primary key (PRODUCT_ID)
    ) tablespace products
    STORAGE (INITIAL 1M NEXT 1M);
```
    

**Parent topic:** [Create Tables Using the VECTOR Data Type](create-tables-using-vector-data-type.md)
