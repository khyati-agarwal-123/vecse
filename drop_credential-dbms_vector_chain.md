## DROP_CREDENTIAL {#GUID-FCA739BA-4849-45BF-A860-251D45E31B17}

Use the `DBMS_VECTOR_CHAIN.DROP_CREDENTIAL` credential helper procedure to drop an existing credential name from the data dictionary. 

Syntax
```
    DBMS_VECTOR_CHAIN.DROP_CREDENTIAL (
    CREDENTIAL_NAME      IN VARCHAR2
    );
```
    

CREDENTIAL_NAME

Specify the credential name that you want to drop.

Examples

  * For Generative AI:
```
    exec dbms_vector_chain.drop_credential('OCI_CRED');
```
    

  * For Cohere:
```
    exec dbms_vector_chain.drop_credential('COHERE_CRED');
```
    




**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
