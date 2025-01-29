## DROP_PREFERENCE {#GUID-72CCB0E1-2FB9-4F94-A3A5-6090E1927D3A}

Use the `DBMS_VECTOR_CHAIN.DROP_PREFERENCE` preference helper procedure to remove an existing Vectorizer preference. 

Syntax
```
    DBMS_VECTOR_CHAIN.DROP_PREFERENCE (PREF_NAME);
```
    

PREF_NAME

Name of the Vectorizer preference to drop.

Example
```
    DBMS_VECTOR_CHAIN.DROP_PREFERENCE ('scott_vectorizer');
```
    

**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
