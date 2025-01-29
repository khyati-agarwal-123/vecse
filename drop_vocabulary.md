## DROP_VOCABULARY {#GUID-2CC9EBDB-717F-4AA7-8FDE-7503BE185D87}

Use the `DBMS_VECTOR_CHAIN.DROP_VOCABULARY` chunker helper procedure to remove vocabulary data from the data dictionary. 

Syntax
```
    DBMS_VECTOR_CHAIN.DROP_VOCABULARY(
    VOCABULARY_NAME    IN VARCHAR2
    );
```
    

VOCAB_NAME

Specify the name of the vocabulary that you want to drop, in the form: *`vocabulary_name`* 

or *`owner.vocabulary_name`* 

Example
```
    DBMS_VECTOR_CHAIN.DROP_VOCABULARY('MY_VOCAB_1');
```
    

**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
