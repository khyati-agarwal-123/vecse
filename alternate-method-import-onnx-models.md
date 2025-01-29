## Alternate Method to Import ONNX Models {#GUID-396E895D-718B-40F5-84C5-F00C120E2606}

Use the `DBMS_DATA_MINING.IMPORT_ONNX_MODEL` procedure to import the model and declare the input name. A PL/SQL helper block is used to facilitate the process of importing the ONNX format model into the Oracle Database in the included example. 

Perform the following steps to import ONNX model into the Database using `DBMS_DATA_MINING` PL/SQL package. 

  * Connect as `dmuser`. 
```
    CONN dmuser/@;
```
    

  * Run the following helper PL/SQL block:
```
    DECLARE
    m_blob BLOB default empty_blob();
    m_src_loc BFILE ;
    BEGIN
    DBMS_LOB.createtemporary (m_blob, FALSE);
    m_src_loc := BFILENAME('DM_DUMP', 'my_embedding_model.onnx');
    DBMS_LOB.fileopen (m_src_loc, DBMS_LOB.file_readonly);
    DBMS_LOB.loadfromfile (m_blob, m_src_loc, DBMS_LOB.getlength (m_src_loc));
    DBMS_LOB.CLOSE(m_src_loc);
    DBMS_DATA_MINING.import_onnx_model ('doc_model', m_blob, JSON('{"function" : "embedding", "embeddingOutput" : "embedding", "input": {"input": ["DATA"]}}'));
    DBMS_LOB.freetemporary (m_blob);
    END;
    /
```
    

The code sets up a `BLOB` object and a `BFILE` locator, creates a temporary `BLOB` for storing the `my_embedding_model.onnx` file from the `DM_DUMP` directory, and reads its contents into the `BLOB`. It then closes the file and uses the content to import an ONNX model into the database with specified metadata, before releasing the temporary `BLOB` resources. 




The schema of the `IMPORT_ONNX_MODEL` procedure is as follows: `DBMS_DATA_MINING.IMPORT_ONNX_MODEL(model_data, model_name, metadata)`. This procedure loads `IMPORT_ONNX_MODEL` from the `DBMS_DATA_MINING` package to import the ONNX model into the Database using the name provided in `model_name`, the BLOB content in `m_blob`, and the associated `metadata`. 

  * `doc_model`: This parameter is a user-specified name under which the imported model is stored in the Oracle Database. 

  * `m_blob`: This is a model data in `BLOB` that holds the ONNX representation of the model. 

  * `"function" : "embedding"`: Indicates the function name for text embedding model. 

  * `"embeddingOutput" : "embedding"`: Specifies the output variable which contains the embedding results. 

  * `"input": {"input": ["DATA"]}`: Specifies a JSON object (`"input"`) that describes the input expected by the model. It specifies that there is an input named `"input"`, and its value should be an array with one element, `"DATA"`. This indicates that the model expects a single string input to generate embeddings. 




Alternately, the `DBMS_DATA_MINING.IMPORT_ONNX_MODEL` procedure can also accept a `BLOB` argument representing an ONNX file stored and loaded from OCI Object Storage. The following is an example to load an ONNX model stored in an OCI Object Storage. 
```
    DECLARE
    model_source BLOB := NULL;
    BEGIN
    -- get BLOB holding onnx model
    model_source := DBMS_CLOUD.GET_OBJECT(
    credential_name => 'myCredential',
    object_uri => 'https://objectstorage.us-phoenix -1.oraclecloud.com/' ||
    'n/namespace -string/b/bucketname/o/myONNXmodel.onnx');
    
    DBMS_DATA_MINING.IMPORT_ONNX_MODEL(
    "myonnxmodel",
    model_source,
    JSON('{ function : "embedding" })
    );
    END;
    /
```
    

This PL/SQL block starts by initializing a `model_source` variable as a `BLOB` type, initially set to NULL. It then retrieves an ONNX model from Oracle Cloud Object Storage using the `DBMS_CLOUD.GET_OBJECT` procedure, specifying the credentials `(OBJ_STORE_CRED`) and the URI of the model. The ONNX model resides in a specific bucket named `bucketname` in this case, and is accessible through the provided URL. Then, the script loads the ONNX model into the `model_source` BLOB. The `DBMS_DATA_MINING.IMPORT_ONNX_MODEL` procedure then imports this model into the Oracle Database as `myonnxmodel`. During the import, a JSON metadata specifies the model's function as `embedding`, for embedding operations. 

See [IMPORT_ONNX_MODEL Procedure](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-BBAAE33A-412B-455A-8FE0-81BE31120300#GUID-BBAAE33A-412B-455A-8FE0-81BE31120300) and [GET_OBJECT Procedure and Function](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-3DB888C9-18C7-4A26-8DA8-EDFB260E2B14) to learn about the PL/SQL procedure. 

Example: Importing a Pretrained ONNX Model to Oracle Database

The following presents a comprehensive step-by-step example of importing ONNX embedding and generating vector embeddings.
```
    conn sys/@pdb as sysdba
    grant db_developer_role to dmuser identified by dmuser;
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
    --Alternately, load the model
    EXECUTE DBMS_DATA_MINING.IMPORT_ONNX_MODEL(
    'my_embedding_model.onnx',
    'doc_model',
    JSON('{"function" : "embedding",
    "embeddingOutput" : "embedding",
    "input": {"input": ["DATA"]}}')
    );
    
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
    

**Parent topic:** [Import ONNX Models into Oracle Database End-to-End Example](import-onnx-models-oracle-database-end-end-example.md)
