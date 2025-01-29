## DBA_VECTOR_HITCOUNTS {#GUID-B68E0376-CBD8-4A54-9A45-3B1D4E50F20B}

The `DBA_VECTOR_HITCOUNTS` view tracks calls to third parties for observability. 

Column Name  |  Data Type  |  Description   
---|---|---  
`USER_ID` | `NUMBER` |  ID of the current user  
`CREDENTIAL_ID` | `NUMBER` |  Credential ID for the third party call used by the user  
`PROVIDER_NAME` | `VARCHAR2(128)` |  Name of the third-party provider  
`METHOD_NAME` | `VARCHAR2(128)` |  Name of the method calling the third-party API(s) (such as `UTL_TO_SUMMARY`, `UTL_TO_EMBEDDING`, and so on)   
`TOTAL_COUNT` | `NUMBER` |  Total number of calls made to the third-party provider  
`LAST_HIT` | `TIMESTAMP(6)` |  Time of the last call to the third-party provider  
  
**Parent topic:** [Text Processing Views](text-processing-views.md)
