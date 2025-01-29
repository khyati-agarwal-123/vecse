## ALL_VECTOR_ABBREV_TOKENS {#GUID-60E6DF25-88B9-4FEB-904F-2DDC4FB7FC75}

The `ALL_VECTOR_ABBREV_TOKENS` view displays a list of abbreviation tokens from all supported languages. 

Column Name | Data Type | Description  
---|---|---  
`ABBREV_OWNER` |  `VARCHAR2(128)` |  Owner of the abbreviation token (for example, `PUBLIC`)   
`ABBREV_LANGUAGE` |  `NUMBER` |  Language ID for the language (for example, `1` for American)   
`ABBREV_TOKEN` |  `NVARCHAR2(255)` |  List of all abbreviation tokens corresponding to each language  
  
**Parent topic:** [Text Processing Views](text-processing-views.md)
