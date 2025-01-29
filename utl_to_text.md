## UTL_TO_TEXT {#GUID-73397E89-92FB-48ED-94BB-1AD960C4EA1F}

Use the `DBMS_VECTOR_CHAIN.UTL_TO_TEXT` chainable utility function to convert an input document (for example, PDF, DOC, JSON, XML, or HTML) to plain text. 

Purpose

To perform a file-to-text transformation by using the Oracle Text component (`CONTEXT`) of Oracle Database. 

Syntax
```
    DBMS_VECTOR_CHAIN.UTL_TO_TEXT (
    DATA          IN CLOB | BLOB,
    PARAMS        IN JSON default NULL
    ) return CLOB;
```
    

DATA

This function accepts the input data type as `CLOB` or `BLOB`. It can read documents from a remote location or from files stored locally in the database tables. 

It returns a plain text version of the document as `CLOB`. 

Oracle Text supports around 150 file types. For a complete list of all the supported document formats, see [*Oracle Text Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-BDD5A962-25F7-43B1-B7C3-37B3A467AC0A). 

PARAMS

Specify the following input parameter in JSON format:
```
    {
    "plaintext" : "true or false",
    "charset"   : "UTF8"
    }
```
    

**Table: Parameter Details**

Parameter | Description  
---|---  
`plaintext` |  Plain text output. <br>The default value for this parameter is `true`, that is, by default the output format is plain text. <br>If you do not want to return the document as plain text, then set this parameter to `false`.   
`charset` |  Character set encoding. <br>Currently, only `UTF8` is supported.   
  
Example
```
    select DBMS_VECTOR_CHAIN.UTL_TO_TEXT (
    t.blobdata,
    json('{
    "plaintext": "true",
    "charset"  : "UTF8"
    }')
    ) from tab t;
```
    

**End-to-end example**: 

To run an end-to-end example scenario using this function, see [Convert File to Text to Chunks to Embeddings Within Oracle Database](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-10D0A76C-2DAE-42BE-A332-3EEF6D91D701). 

**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
