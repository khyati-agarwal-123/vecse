## 10Supported Clients and Languages {#GUID-BAD95D81-BD27-4122-A5B6-7B4807EDD63B}

For more information about Oracle AI Vector Search support using some of Oracle's available clients and languages, see the included reference material.

Clients and Languages | Reference Material  
---|---  
PL/SQL | [*Oracle Database PL/SQL Language Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=LNPLS-GUID-160C5139-EDBE-40BE-8DB4-1CA4E8A1CA46)  
MLE JavaScript | [*Oracle Database JavaScript Developer's Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=MLEJS-GUID-85AADE3B-5483-40AE-9844-95ABE3E76EEC)  
JDBC | [*Oracle Database JDBC Developer’s Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=JJDBC-GUID-827F7012-7E00-4510-8E34-1EC07D8E6359)  
Node.js | [node-oracledb documentation](https: <br>//node-oracledb.readthedocs.io/en/latest/user_guide/vector_data_type.md)  
Python | [python-oracledb documentation](https: <br>//python-oracledb.readthedocs.io/en/latest/user_guide/vector_data_type.md)  
Oracle Call Interface | [*Oracle Call Interface Developer's Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=LNOCI-GUID-9EED9287-82F1-4622-8FFF-6E830507C85D)  
ODP.NET | [*Oracle Data Provider for .NET Developer's Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ODPNT-GUID-82C09283-1643-4C75-B836-BFB7FB33E34B)  
SQL*Plus | [*SQL*Plus User's Guide and Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SQPUG-GUID-177F24B7-D154-4F8B-A05B-7568079800C6)  
  
Oracle Database 23ai supports binding with native VECTOR types for all Oracle clients. Applications that use earlier Oracle Client 23ai libraries can connect to Oracle Database 23ai in the following ways:

  * Using the `TO_VECTOR()` SQL function to insert vector data, as shown in the following example: 
```
    INSERT INTO vecTab VALUES(TO_VECTOR('[1.1, 2.9, 3.14]'));
```
    

  * Using the `FROM_VECTOR()` SQL function to fetch vector data, as shown in the following example:
```
    SELECT FROM_VECTOR(dataVec) FROM vecTab;
```
    



