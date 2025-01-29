## Convert Text String to Embedding Within Oracle Database {#GUID-1F9E9C73-6118-427E-8FB7-D4EBCCECC6D0}

Perform a text-to-embedding transformation by accessing a vector embedding model stored in the database.

You can download an embedding machine learning model, convert it into ONNX format (if not already in ONNX format), and load the model into Oracle Database. You can then access that model to vectorize your data that is used to populate a vector index. Note that you must use the same embedding model on both the data to be indexed and the user's input query. In this example, you can see how to vectorize a user's input query on the fly.

Here, you can call either the `VECTOR_EMBEDDING` SQL function or the `UTL_TO_EMBEDDING` PL/SQL function (note the singular "embedding"). Both `VECTOR_EMBEDDING` and `UTL_TO_EMBEDDING` directly return a `VECTOR` type (not an array). 

To convert a user's input text string "`hello`" to a vector embedding, using an embedding model in ONNX format: 

  1. Connect to Oracle Database as a local user.
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
        DROP USER docuser cascade;
```
```
        CREATE USER docuser identified by docuser DEFAULT TABLESPACE tbs1 quota unlimited on tbs1;
```
```
        GRANT DB_DEVELOPER_ROLE, create credential to docuser;
```
        

    3. Connect as the local user (`docuser`):
```
        CONN docuser/password
```
        

  2. Call either `VECTOR_EMBEDDING` or `UTL_TO_EMBEDDING`.

    1. Load your ONNX format embedding model into Oracle Database.

For detailed information on how to perform this step, see [Import ONNX Models into Oracle Database End-to-End Example](import-onnx-models-oracle-database-end-end-example.md#GUID-6AEA7A0E-78E0-4083-A126-4516EB98175A). 

    2. Call `VECTOR_EMBEDDING` or `UTL_TO_EMBEDDING`. You can use `UTL_TO_EMBEDDING` either from the `DBMS_VECTOR` or the `DBMS_VECTOR_CHAIN` package, depending on your use case. 

       * `VECTOR_EMBEDDING`: 
```
            SELECT TO_VECTOR(VECTOR_EMBEDDING(doc_model USING 'hello' as data)) AS embedding;
```
            

       * `DBMS_VECTOR.UTL_TO_EMBEDDING`: 
```
            var params clob;
            exec :params := '{"provider":"database", "model":"doc_model"}';
            
            select dbms_vector.utl_to_embedding('hello', json(:params)) from dual;
```
            

Here, `doc_model` specifies the name under which your embedding model is stored in the database. 

The generated embedding appears as follows:
```
    EMBEDDING
    -------------------------------------------------------------------------------------------------------------------------------------
    [8.78423732E-003,-4.29633334E-002,-5.93001908E-003,-4.65480909E-002,2.14333013E-002,6.53376281E-002,-5.93746938E-002,2.10403297E-002,
    4.38376889E-002,5.22960871E-002,1.25104953E-002,6.49512559E-002,-9.26998071E-003,-6.97442219E-002,-3.02916039E-002,-4.74979728E-003,
    -1.08755399E-002,-4.63751052E-003,3.62781435E-002,-9.35919806E-002,-1.13934642E-002,-5.74270077E-002,-1.36667723E-002,2.42995787E-002,
    -6.96804151E-002,4.93822657E-002,1.01460628E-002,-1.56464987E-002,-2.39410568E-002,-4.27529104E-002,-5.65665103E-002,-1.74160264E-002,
    5.05326502E-002,4.31500375E-002,-2.6994409E-002,-1.72731467E-002,9.30535868E-002,6.85951149E-004,5.61876409E-003,-9.0233935E-003,
    -2.55788807E-002,-2.04174276E-002,3.74175981E-002,-1.67872179E-002,1.07479304E-001,-6.64602639E-003,-7.65537247E-002,-9.71965566E-002,
    -3.99636962E-002,-2.57076006E-002,-5.62455431E-002,-1.3583754E-001,3.45946029E-002,1.85191762E-002,3.01524661E-002,-2.62163244E-002,
    -4.05582506E-003,1.72979087E-002,-3.66434865E-002,-1.72491539E-002,3.95228416E-002,-1.05518714E-001,-1.27463877E-001,1.42578809E-002
```
    




**Related Topics**

  * [UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector.md#GUID-8E615832-F6C0-4435-8F43-3FAF80692D5B)
  * [VECTOR_EMBEDDING](vector_embedding.md#GUID-5ED78260-6D21-4B6B-86E0-A1E70EFA11CA)



**Parent topic:** [Generate Embeddings](generate-embeddings.md)
