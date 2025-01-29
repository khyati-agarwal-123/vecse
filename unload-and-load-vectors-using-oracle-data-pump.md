## Unload and Load Vectors Using Oracle Data Pump {#GUID-28A39166-0345-467F-99C3-CA6BAD4CF277}

Starting with Oracle Database 23ai, Oracle Data Pump enables you to use multiple components to load and unload vectors to databases. 

Oracle Data Pump technology enables very high-speed movement of data and metadata from one database to another. Oracle Data Pump is made up of three distinct components: Command-line clients, expdp and impdp; the DBMS_DATAPUMP PL/SQL package (also known as the Data Pump API); and the DBMS_METADATA PL/SQL package (also known as the Metadata API). 

Unloading and Loading a table with vector datatype columns is supported in all modes (`FULL`, `SCHEMA`, `TABLES`) using all the available access methods (`DIRECT_PATH`,` EXTERNAL_TABLE`, `AUTOMATIC`, `INSERT_AS_SELECT`). 

Examples Vector Export and Import Syntax
```
    expdp /@  dumpfile=.dmp directory= full=y metrics=y access_method=direct_path
    
    expdp /@  dumpfile=.dmp directory= schemas= metrics=y access_method=external_table
    
    expdp /@  dumpfile=.dmp directory= tables=. metrics=y access_method=direct_path
    
    impdp /@  dumpfile=.dmp directory= metrics=y access_method=direct_path
```
    

> **note:** 

  * `TABLE_EXISTS_ACTION=APPEND | TRUNCATE` can only be used with the `EXTERNAL_TABLE` access method. 
  * `TABLE_EXISTS_ACTION=APPEND | TRUNCATE` can load `VECTOR` column data into a `VARCHAR2` column if the conversion can fit into that `VARCHAR2`. 
  * `TABLE_EXISTS_ACTION=APPEND | TRUNCATE` can only load a `VECTOR` column with the source `VECTOR` data dimasion that matches that loaded `VECTOR` column's dimension. If the dimension does not match, then an error is raised. 
  * `TABLE_EXISTS_ACTION=REPLACE` supports any access method. 
  * It is not possible to use a the transportable tablespace mode with vector indexes. However, this mode supports tables with the `VECTOR` datatype. 



**Related Topics**

  * [Overview of Oracle Data Pump](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SUTIL-GUID-17FAE261-0972-4220-A2E4-44D479F519D4)
  * [DBMS_DATAPUMP](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-AEA7ED80-DB4A-4A70-B199-592287206348)
  * [DBMS_METADATA](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-F72B5833-C14E-4713-A588-6BDF4D4CBA2A)



**Parent topic:** [Store Vector Embeddings](store-vector-embeddings.md)
