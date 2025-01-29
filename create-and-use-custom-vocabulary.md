## Create and Use Custom Vocabulary {#GUID-B6527DDC-8EF0-479E-8965-6C2459E7827A}

Create and use your own vocabulary of tokens when chunking data.

Here, you use the chunker helper function `CREATE_VOCABULARY` from the `DBMS_VECTOR_CHAIN` package to load custom vocabulary. This vocabulary file contains a list of tokens, recognized by your vector embedding model's tokenizer. 

  1. Connect as a local user and prepare your data dump directory.
    1. Log in to SQL*Plus as the `SYS` user, connecting as `SYSDBA`:
```
        conn sys/password as sysdba
```
```
        CREATE TABLESPACE tbs1
        DATAFILE 'tbs5.dbf' SIZE 20G AUTOEXTEND ON
        EXTENT MANAGEMENT LOCAL
        SEGMENT SPACE MANAGEMENT AUTO;
```
```
        SET ECHO ON
        SET FEEDBACK 1
        SET NUMWIDTH 10
        SET LINESIZE 80
        SET TRIMSPOOL ON
        SET TAB OFF
        SET PAGESIZE 10000
        SET LONG 10000
```
        

    2. Create a local user (`docuser`) and grant necessary privileges:
```
        drop user docuser cascade;
```
```
        create user docuser identified by docuser DEFAULT TABLESPACE tbs1 quota unlimited on tbs1;
```
```
        grant DB_DEVELOPER_ROLE to docuser;
```
        

    3. Create a local directory (`VEC_DUMP`) to store your vocabulary file. Grant necessary privileges:
```
        create or replace directory VEC_DUMP as '/my_local_dir/';
```
```
        grant read, write on directory VEC_DUMP to docuser;
        
        commit;
```
        

    4. Transfer the vocabulary file for your required model to the `VEC_DUMP` directory. 

For example, if using the WordPiece tokenization, you can download and transfer the `vocab.txt` vocabulary file for "bert-base-uncased". 

    5. Connect as the local user (`docuser`):
```
        conn docuser/password
```
        

  2. Create a relational table (`doc_vocabtab`) to store your vocabulary tokens in it:
```
    CREATE TABLE doc_vocabtab(token nvarchar2(64))
    ORGANIZATION EXTERNAL
    (default directory VEC_DUMP
    ACCESS PARAMETERS (RECORDS DELIMITED BY NEWLINE)
    location ('bert-vocabulary-uncased.txt'));
```
    

  3. Create a vocabulary (`doc_vocab`) by calling `DBMS_VECTOR_CHAIN.CREATE_VOCABULARY`:
```
    DECLARE
    vocab_params clob := '{
    "table_name"      : "doc_vocabtab",
    "column_name"     : "token",
    "vocabulary_name" : "doc_vocab",
    "format"          : "bert",
    "cased"           : false
    }';
    
    BEGIN
    dbms_vector_chain.create_vocabulary(json(vocab_params));
    END;
    /
```
    




After loading the token vocabulary, you can now use the `BY VOCABULARY` chunking mode (with `VECTOR_CHUNKS` or `UTL_TO_CHUNKS`) to split data by counting the number of tokens. 

**Related Topics**

  * [CREATE_VOCABULARY](create_vocabulary.md#GUID-2D19528E-0F0D-4102-8EC7-E9EA62C66C2D)
  * [VECTOR_CHUNKS](vector_chunks.md#GUID-5927E2FA-6419-4744-A7CB-3E62DBB027AD)
  * [UTL_TO_CHUNKS](utl_to_chunks-dbms_vector_chain.md#GUID-4E145629-7098-4C7C-804F-FC85D1F24240)



**Parent topic:** [Configure Chunking Parameters](configure-chunking-parameters.md)
