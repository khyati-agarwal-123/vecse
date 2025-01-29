## USER_VECTOR_HITCOUNTS {#GUID-BDC2946B-ED01-493A-9688-19E2920112BB}

The `USER_VECTOR_HITCOUNTS` view tracks calls to third parties for observability for the current user. 

Column Name  |  Data Type  |  Description   
---|---|---  
`USER_ID` | `NUMBER` |  ID of the current user  
`CREDENTIAL_ID` | `NUMBER` |  Credential ID for the third-party call used by the user  
`CREDENTIAL_NAME` | `VARCHAR2(128)` |  Credential name for the third-party call used by the user  
`PROVIDER_NAME` | `VARCHAR2(128)` |  Name of the third-party provider  
`METHOD_NAME` | `VARCHAR(128)` |  Name of the method calling the third-party API(s) (such as `UTL_TO_SUMMARY`, `UTL_TO_EMBEDDING`, and so on)   
`TOTAL_COUNT` | `NUMBER` |  Total number of calls made to the third-party provider.  
`LAST_HIT` | `TIMESTAMP(6)` |  Time of the last call to the third-party provider  
  
**Parent topic:** [Text Processing Views](text-processing-views.md)
