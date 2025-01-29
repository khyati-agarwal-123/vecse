## Vectors in External Tables {#GUID-BFD34A9A-8E2F-425B-B68A-15BF630D937D}

External tables can be created with columns of type `VECTOR`, allowing vector embeddings represented in text or binary format stored in external files to be rendered as the `VECTOR` data type in Oracle Database. 

The ability to store `VECTOR` data in external tables can be beneficial when considering the vast amount of data that is involved in AI workflows. Using external tables provides a convenient option to store vector embeddings and contextual information outside of the database while still using the database as a semantic search engine. 

The following types of external files support `VECTOR` columns in external tables: 

  * CSV
  * Parquet
  * Avro
  * ORC
  * Dmp



External tables with `VECTOR` columns can be accessed by using the drivers `ORACLE_LOADER`, `ORACLE_DATAPUMP`, and `ORACLE_BIGDATA`. Once the chosen driver has loaded the file data into the database, you can interact with the external table rows in any SQL operation (supported by your preferred SQL interface), such as joins, SQL functions, aggregation, and so on. 

Vectors of any dimension format and storage format are supported. If a vector is `SPARSE`, the data must be provided as text as opposed to an array or list format. 

Columns of type `VECTOR` can be included in both explicitly created external tables as well as inline external tables created as part of a `SELECT` statement. The benefit of this approach is that there is no need to predefine a static table to access the vectors in the external table before loading them into the database. The external tables mapping is persisted only while the external table is in use by the SQL query. For an example, see [Querying an Inline External Table](querying-inline-external-table.md#GUID-9D1D2768-77D9-49CC-8FC7-8474E1C6E294), which shows how the external table mappings are created as part of the SQL query operation. Once the query has completed, the external table mapping is discarded from the database. 

Additionally, the `row_limiting_clause` can be used in `SELECT` statements that reference external tables. Internal and external tables can be referenced in the same query. You can use the `CREATE TABLE AS SELECT` statement to create an internal table by selecting from an external table with `VECTOR` column(s). Similarly, you can use the `INSERT INTO SELECT` statement to insert values into an internal table from an external table called in the `SELECT` subquery. 

> **note:** 

  * External tables are not currently supported in multi-vector similarity searches.
  * HNSW and IVF indexes cannot currently be created on `VECTOR` columns stored in external tables. 



Vector embeddings in external tables can be accessed for use in similarity searches in the same way as you would use an internal table, as in the following query:
```
    SELECT id, embedding
    FROM external_table
    ORDER BY VECTOR_DISTANCE(embedding, '[1,1]', COSINE)
    FETCH APPROX FIRST 3 ROWS ONLY WITH TARGET ACCURACY 90;
```
    

The following examples demonstrate the syntax used to create external tables with `VECTOR` columns depending on the access driver: 

  * Using `ORACLE_LOADER`:
```
    CREATE TABLE ext_vec_tab1(
    v1 VECTOR,
    v2 VECTOR
    )
    ORGANIZATION EXTERNAL
    (
    TYPE ORACLE_LOADER
    DEFAULT DIRECTORY my_dir
    ACCESS PARAMETERS
    (
    RECORDS DELIMITED BY NEWLINE
    FIELDS TERMINATED BY ":"
    MISSING FIELD VALUES ARE NULL
    )
    LOCATION('my_ext_vec_embeddings.csv')
    )
    REJECT LIMIT UNLIMITED;
```
    

  * Using `ORACLE_DATAPUMP`:
```
    -- First create the table with the loader
    CREATE TABLE dp_ext_tab(
    country_code         VARCHAR2(5),
    country_name         VARCHAR2(50),
    country_language     VARCHAR2(50),
    country_vector       VECTOR(*,*)
    )
    ORGANIZATION EXTERNAL
    (
    TYPE ORACLE_LOADER
    DEFAULT DIRECTORY my_dir
    ACCESS PARAMETERS
    (
    RECORDS DELIMITED BY NEWLINE
    FIELDS TERMINATED BY ":"
    MISSING FIELD VALUES ARE NULL
    (
    country_code         CHAR(5),
    country_name         CHAR(50),
    country_language     CHAR(50),
    country_vector       CHAR(10000)
    )
    )
    LOCATION ('ext_vectorcountries.dat')
    )
    PARALLEL 5
    REJECT LIMIT UNLIMITED;
    
    -- Then generate the dmp file
    CREATE TABLE ext_export_table
    ORGANIZATION EXTERNAL
    (
    TYPE ORACLE_DATAPUMP
    DEFAULT DIRECTORY my_dir
    LOCATION ('ext.dmp')
    )
    AS SELECT * FROM dp_ext_tab;
    
    -- Finally, create an external table with the datapump driver
    CREATE TABLE dp_ext_tab_final
    (
    country_code         VARCHAR2(5),
    country_name         VARCHAR2(50),
    country_language     VARCHAR2(50),
    country_vector       VECTOR(3, INT8)
    )
    ORGANIZATION EXTERNAL
    (
    TYPE ORACLE_DATAPUMP
    DEFAULT DIRECTORY my_dir
    LOCATION ('ext.dmp')
    )
    PARALLEL 5
    REJECT LIMIT UNLIMITED;
```
    

  * Using `ORACLE_BIGDATA`:
```
    CREATE TABLE bd_ext_tab
    (
    COL1 vector(5,INT8),
    COL2 vector(5,INT8),
    COL3 vector(5,INT8),
    COL4 vector(5,INT8)
    )
    ORGANIZATION external
    (
    TYPE ORACLE_BIGDATA
    DEFAULT DIRECTORY my_dir
    ACCESS PARAMETERS
    (
    com.oracle.bigdata.credential.name\=OCI_CRED
    com.oracle.bigdata.credential.schema\=PDB_ADMIN
    com.oracle.bigdata.fileformat=parquet
    com.oracle.bigdata.debug=true
    )
    LOCATION ( 'https://swiftobjectstorage.<region>.oraclecloud.com/v1/<namespace>/<filepath>/basic_vec_data.parquet' )
    )
    REJECT LIMIT UNLIMITED PARALLEL 2;
```
    




  * [Querying an Inline External Table](querying-inline-external-table.md)  
In this example, vectors in an external table of type `ORACLE_BIGDATA` are queried as part of a vector search. 
  * [Performing a Semantic Similarity Search Using External Table](performing-semantic-similarity-search-using-external-table.md)  
See a SQL example plan that illustrates how you can use external tables as the data set for semantic similarity searches 



> **note:** See Also: 

  * [*Oracle Database Utilities*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SUTIL-GUID-44323E01-7D72-45EC-915A-99E596769D9E) for more information about external tables 



**Parent topic:** [Create Tables Using the VECTOR Data Type](create-tables-using-vector-data-type.md)
