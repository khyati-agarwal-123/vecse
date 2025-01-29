## DROP_LANG_DATA {#GUID-55F54E07-1026-4021-A194-3E068471426E}

Use the `DBMS_VECTOR_CHAIN.DROP_LANG_DATA` chunker helper procedure to remove abbreviation data from the data dictionary. 

Syntax
```
    DBMS_VECTOR_CHAIN.DROP_LANG_DATA(
    PREF_NAME     IN VARCHAR2
    );
```
    

LANG

Specify the name of the language data that you want to drop for a given language. 

Example
```
    DBMS_VECTOR_CHAIN.DROP_LANG_DATA('indonesian');
```
    

**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
