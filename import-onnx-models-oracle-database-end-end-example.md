## Import ONNX Models into Oracle Database End-to-End Example {#GUID-6AEA7A0E-78E0-4083-A126-4516EB98175A}

Learn to import a pretrained embedding model that is in ONNX format and generate vector embeddings.

Follow the steps below to import a pretrained ONNX formatted embedding model into the Oracle Database.

Prepare Your Data Dump Directory

Prepare your data dump directory and provide the necessary access and privileges to `dmuser`. 

  1. Choose from: 
    1. If you already have a pretrained ONNX embedding model, store it in your working folder.

    2. If you do not have pretrained embedding model in ONNX format, perform the steps listed in [Convert Pretrained Models to ONNX Format](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=MLPUG-GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6). 

  2. Login to SQL*Plus as `SYSDBA` in your PDB. 
```
    CONN sys/@pdb as sysdba;
```
    

  3. Grant the `DB_DEVELOPER_ROLE` to `dmuser`. 
```
    GRANT DB_DEVELOPER_ROLE TO dmuser identified by ;
```
    

  4. Grant `CREATE MINING MODEL` privilege to `dmuser`. 
```
    GRANT create mining model TO dmuser;
```
    

  5. Set your working folder as the data dump directory (`DM_DUMP`) to load the ONNX embedding model. 
```
    CREATE OR REPLACE DIRECTORY DM_DUMP as '';
```
    

  6. Grant `READ` permissions on the `DM_DUMP` directory to `dmuser`. 
```
    GRANT READ ON DIRECTORY dm_dump TO dmuser;
```
    

  7. Grant `WRITE` permissions on the `DM_DUMP` directory to `dmuser`. 
```
    GRANT WRITE ON DIRECTORY dm_dump TO dmuser;
```
    

  8. Drop the model if it already exits.
```
    exec DBMS_VECTOR.DROP_ONNX_MODEL(model_name => 'doc_model', force => true);
```
    




Import ONNX Model Into the Database

You created a data dump directory and now you load the ONNX model into the Database. Use the `DBMS_VECTOR.LOAD_ONNX_MODEL` procedure to load the model. The `DBMS_VECTOR.LOAD_ONNX_MODEL` procedure facilitates the process of importing ONNX format model into the Oracle Database. In this example, the procedure loads an ONNX model file, named `my_embedding_model.onnx` from the `DM_DUMP` directory, into the Database as `doc_model`, specifying its use for embedding tasks. 

  1. Connect as `dmuser`. 
```
    CONN dmuser/@;
```
    

  2. Load the ONNX model into the Database. 

If the ONNX model to be imported already includes an output tensor named `embeddingOutput` and an input string tensor named `data`, JSON metadata is unnecessary. Embedding models converted from OML4Py follow this convention and can be imported without the JSON metadata. 
```
    EXECUTE DBMS_VECTOR.LOAD_ONNX_MODEL(
    'DM_DUMP',
    'my_embedding_model.onnx',
    'doc_model');
```
    

Alternately, you can load the ONNX embedding model by specifying the JSON metadata.
```
    EXECUTE DBMS_VECTOR.LOAD_ONNX_MODEL(
    'DM_DUMP',
    'my_embedding_model.onnx',
    'doc_model',
    JSON('{"function" : "embedding", "embeddingOutput" : "embedding", "input": {"input": ["DATA"]}}'));
```
    




The procedure `LOAD_ONNX_MODEL` declares these parameters: 

  * `DM_DUMP`: specifies the directory name of the data dump. 

> **note:** Ensure that the `DM_DUMP` directory is defined. 

  * `my_embedding_model`: is a `VARCHAR2` type parameter that specifies the name of the ONNX model. 

  * `doc_model`: This parameter is a user-specified name under which the model is stored in the Oracle Database. 

  * The JSON metadata associated with the ONNX model is declared as:

`"function" : "embedding"`: Indicates the function name for text embedding model. 

`"embeddingOutput" : "embedding"`: Specifies the output variable which contains the embedding results. 

  * `"input": {"input": ["DATA"]}`: Specifies a JSON object (`"input"`) that describes the input expected by the model. It specifies that there is an input named `"input"`, and its value should be an array with one element, `"DATA"`. This indicates that the model expects a single string input to generate embeddings. 




For more information about the `LOAD_ONNX_MODEL` procedure, see [*Oracle Database PL/SQL Packages and Types Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-7F1D7992-D8F7-4AD9-9BF6-6EFFC1B0617A). 

Alternatively, if your ONNX embedding model is loaded on cloud object storage, the `LOAD_ONNX_MODEL_CLOUD` procedure can be used. For more information, see [*Oracle Database PL/SQL Packages and Types Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-82A8D291-8096-4A7C-8882-9B6AC4A7FCCB). 

Query Model Statistics

You can view model attributes and learn about the model by querying machine learning dictionary views and model detail views. 

> **note:** *`DOC_MODEL`* is the user-specified name of the embedding text model. 

  1. Query `USER_MINING_MODEL_ATTRIBUTES` view. 
```
    SELECT model_name, attribute_name, attribute_type, data_type, vector_info
    FROM user_mining_model_attributes
    WHERE model_name = 'DOC_MODEL'
    ORDER BY ATTRIBUTE_NAME;
```
    

To learn about `USER_MINING_MODEL_ATTRIBUTES` view, see [USER_MINING_MODEL_ATTRIBUTES](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=REFRN-GUID-A8B668BE-01CB-45B0-B91F-89545B58821B#GUID-A8B668BE-01CB-45B0-B91F-89545B58821B). 

  2. Query `USER_MINING_MODELS` view. 
```
    SELECT MODEL_NAME, MINING_FUNCTION, ALGORITHM,
    ALGORITHM_TYPE, MODEL_SIZE
    FROM user_mining_models
    WHERE model_name = 'DOC_MODEL'
    ORDER BY MODEL_NAME;
```
    

To learn about `USER_MINING_MODELS` view, see [USER_MINING_MODELS](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=REFRN-GUID-A43502EF-129D-473F-A19B-72503A6886BF#GUID-A43502EF-129D-473F-A19B-72503A6886BF). 

  3. Check model statistics by viewing the model detail views. Query the `DM$VM*`DOC_MODEL`*` view. 
```
    SELECT * FROM DM$VMDOC_MODEL ORDER BY NAME;
```
    

To learn about model details views for ONNX embedding models, see [Model Details Views for ONNX Models](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=DMPRG-GUID-DC0E3481-8E51-455E-99C6-1CBA42389D9D#GUID-DC0E3481-8E51-455E-99C6-1CBA42389D9D). 

  4. Query the `DM$VP*`DOC_MODEL`*` model detail view. 
```
    SELECT * FROM DM$VPDOC_MODEL ORDER BY NAME;
```
    

  5. Query the `DM$VJ*`DOC_MODEL`*` model detail view. 
```
    SELECT * FROM DM$VJDOC_MODEL;
```
    




Generate Embeddings

Apply the model and generate vector embeddings for your input. Here, the input is *hello*. 

Generate vector embeddings using the `VECTOR_EMBEDDING` function. 
```
    SELECT TO_VECTOR(VECTOR_EMBEDDING(doc_model USING 'hello' as data)) AS embedding;
```
    

To learn about the `VECTOR_EMBEDDING` SQL function, see [VECTOR_EMBEDDING](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SQLRF-GUID-5ED78260-6D21-4B6B-86E0-A1E70EFA11CA). You can use the `UTL_TO_EMBEDDING` function in the `DBMS_VECTOR_CHAIN` PL/SQL package to generate vector embeddings generically through REST endpoints. To explore these functions, see the example [Convert Text String to Embedding](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-1F9E9C73-6118-427E-8FB7-D4EBCCECC6D0). 

Example: Importing a Pretrained ONNX Model to Oracle Database

The following presents a comprehensive step-by-step example of importing ONNX embedding and generating vector embeddings.
```
    conn sys/@pdbname as sysdba
    grant db_developer_role to dmuser identified by ;
    grant create mining model to dmuser;
    
    create or replace directory DM_DUMP as '';
    grant read on directory dm_dump to dmuser;
    grant write on directory dm_dump to dmuser;
    >conn dmuser/@;
    
    –- Drop the model if it exits
    exec DBMS_VECTOR.DROP_ONNX_MODEL(model_name => 'doc_model', force => true);
    
    -- Load Model
    EXECUTE DBMS_VECTOR.LOAD_ONNX_MODEL(
    'DM_DUMP',
    'my_embedding_model.onnx',
    'doc_model',
    JSON('{"function" : "embedding", "embeddingOutput" : "embedding"}'));
    /
    
    --check the attributes view
    set linesize 120
    col model_name format a20
    col algorithm_name format a20
    col algorithm format a20
    col attribute_name format a20
    col attribute_type format a20
    col data_type format a20
    
    SQL> SELECT model_name, attribute_name, attribute_type, data_type, vector_info
    FROM user_mining_model_attributes
    WHERE model_name = 'DOC_MODEL'
    ORDER BY ATTRIBUTE_NAME;
    
    
    OUTPUT:
    
    MODEL_NAME           ATTRIBUTE_NAME       ATTRIBUTE_TYPE       DATA_TYPE  VECTOR_INFO
    -------------------- -------------------- -------------------- ---------- ---------------
    DOC_MODEL                INPUT_STRING         TEXT                 VARCHAR2
    DOC_MODEL                ORA$ONNXTARGET       VECTOR               VECTOR     VECTOR(128,FLOA
    T32)
    
    
    
    SQL> SELECT MODEL_NAME, MINING_FUNCTION, ALGORITHM,
    ALGORITHM_TYPE, MODEL_SIZE
    FROM user_mining_models
    WHERE model_name = 'DOC_MODEL'
    ORDER BY MODEL_NAME;
    
    OUTPUT:
    MODEL_NAME           MINING_FUNCTION                ALGORITHM            ALGORITHM_ MODEL_SIZE
    -------------------- ------------------------------ -------------------- ---------- ----------
    DOC_MODEL                EMBEDDING                      ONNX                 NATIVE       17762137
    
    
    
    SQL> select * from DM$VMDOC_MODEL ORDER BY NAME;
    
    OUTPUT:
    NAME                                     VALUE
    ---------------------------------------- ----------------------------------------
    Graph Description                        Graph combining g_8_torch_jit and torch_
    jit
    g_8_torch_jit
    
    
    
    torch_jit
    
    
    Graph Name                               g_8_torch_jit_torch_jit
    Input[0]                                 input:string[1]
    Output[0]                                embedding:float32[?,128]
    Producer Name                            onnx.compose.merge_models
    Version                                  1
    
    6 rows selected.
    
    
    SQL> select * from DM$VPDOC_MODEL ORDER BY NAME;
    
    OUTPUT:
    NAME                                     VALUE
    ---------------------------------------- ----------------------------------------
    batching                                 False
    embeddingOutput                          embedding
    
    
    SQL> select * from DM$VJDOC_MODEL;
    
    OUTPUT:
    METADATA
    --------------------------------------------------------------------------------
    {"function":"embedding","embeddingOutput":"embedding","input":{"input":["DATA"]}}
    
    
    
    --apply the model
    SQL> SELECT TO_VECTOR(VECTOR_EMBEDDING(doc_model USING 'hello' as data)) AS embedding;
    
    --------------------------------------------------------------------------------
    [-9.76553112E-002,-9.89954844E-002,7.69771636E-003,-4.16760892E-003,-9.69305634E-002,
    -3.01141385E-002,-2.63396613E-002,-2.98553891E-002,5.96499592E-002,4.13885899E-002,
    5.32859489E-002,6.57707453E-002,-1.47056757E-002,-4.18472625E-002,4.1588001E-002,
    -2.86354572E-002,-7.56499246E-002,-4.16395674E-003,-1.52879998E-001,6.60010576E-002,
    -3.9013084E-002,3.15719917E-002,1.2428958E-002,-2.47651711E-002,-1.16851285E-001,
    -7.82847106E-002,3.34323719E-002,8.03267583E-002,1.70483496E-002,-5.42407483E-002,
    6.54291287E-002,-4.81935125E-003,6.11041225E-002,6.64106477E-003,-5.47
```
    

**Oracle AI Vector Search SQL Scenario**

To learn how you can chunk *`database-concepts23ai.pdf`* and *`oracle-ai-vector-search-users-guide.pdf`*, generate vector embeddings, and perform similarity search using vector indexes, see [Quick Start SQL](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-403EB84E-3047-4905-844C-BD4A8670B8A4). 

  * [Alternate Method to Import ONNX Models](alternate-method-import-onnx-models.md)  
Use the `DBMS_DATA_MINING.IMPORT_ONNX_MODEL` procedure to import the model and declare the input name. A PL/SQL helper block is used to facilitate the process of importing the ONNX format model into the Oracle Database in the included example. 



**Parent topic:** [Import Pretrained Models in ONNX Format](import-pretrained-models-onnx-format-vector-generation-database.md)
