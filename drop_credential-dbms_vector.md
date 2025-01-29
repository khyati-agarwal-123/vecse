## DROP_CREDENTIAL {#GUID-9F5DA186-6F2E-471F-9CEC-B32D502A1695}

Use the `DBMS_VECTOR.DROP_CREDENTIAL` credential helper procedure to drop an existing credential name from the data dictionary. 

Syntax
```
    DBMS_VECTOR.DROP_CREDENTIAL (
    CREDENTIAL_NAME      IN VARCHAR2
    );
```
    

CREDENTIAL_NAME

Specify the credential name that you want to drop.

Examples

  * For Generative AI:
```
    exec dbms_vector.drop_credential('OCI_CRED');
```
    

  * For Cohere:
```
    exec dbms_vector.drop_credential('COHERE_CRED');
```
    




**Parent topic:** [DBMS_VECTOR](dbms_vector-vecse.md)
