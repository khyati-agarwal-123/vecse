## LOAD_ONNX_MODEL {#GUID-7F1D7992-D8F7-4AD9-9BF6-6EFFC1B0617A}

This procedure enables you to load an ONNX model into the Database.

Syntax
```
    DBMS_VECTOR.LOAD_ONNX_MODEL (
    directory  VARCHAR2,
    file_name   VARCHAR2,
    model_name  VARCHAR2,
    metadata   JSON);
```
```
    DBMS_VECTOR.LOAD_ONNX_MODEL(
    model_name  IN  VARCHAR2,
    model_data  IN  BLOB,
    metadata    IN  JSON);
```
    

Parameters

**Table: LOAD_ONNX_MODEL Procedure Parameters**

Parameter  |  Description   
---|---  
`directory` |  The directory name of the data dump. For example, `DM_DUMP`.   
`file_name` |  A `VARCHAR2` type parameter that specifies the name of the ONNX model.   
`model_name` |  Name of the model in the form `[schema_name.]model_name`. If you do not specify a schema, then your own schema is used.   
`model_data` |  It is a `BLOB` holding the ONNX representation of the model. The `BLOB` contains the identical byte sequence as the one stored in an ONNX file.   
`metadata` | A JSON description of the metadata describing the model. The metadata at minimum must describe the machine learning function supported by the model. The model's metadata parameters are described in [JSON Metadata Parameters for ONNX Models](json-metadata-parameters-onnx-models.md#GUID-B6D784CE-0A52-43B5-A92C-231358416A4B).   
  
Examples

The following examples illustrates a code snippet of using the `DBMS_VECTOR.LOAD_ONNX_MODEL` procedure. The complete step-by-step example is illustrated in [Import ONNX Models and Generate Embeddings](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=DMCON-GUID-6AEA7A0E-78E0-4083-A126-4516EB98175A#GUID-6AEA7A0E-78E0-4083-A126-4516EB98175A). 
```
    EXECUTE DBMS_VECTOR.LOAD_ONNX_MODEL(
    'DM_DUMP',
    'my_embedding_model.onnx',
    'doc_model',
    JSON('{"function" : "embedding", "embeddingOutput" : "embedding", "input": {"input": ["DATA"]}}'));
```
```
    DBMS_VECTOR.LOAD_ONNX_MODEL('my_embedding_model.onnx',
    :blob_bind_variable,
    JSON('{"function" : "embedding",
    "embeddingOutput" : "embedding" ,
    "input":{"input": ["DATA"]}}'));
```
    

For a complete example to illustrate how you can define a `BLOB` variable and use it in the `LOAD_ONNX_MODEL` procedure, you can have the following: 
```
    CREATE OR REPLACE MY_LOAD_EMBEDDING_MODEL(embedding_model_name VARCHAR2, onnx_blob BLOB) IS
    BEGIN
    DBMS_VECTOR.LOAD_ONNX_MODEL(embedding_model_name,
    onnx_blob,
    JSON('{"function" : "embedding",
    "embeddingOutput" : "embedding" ,
    "input":{"input": ["DATA"]}}'));
    END;
    /
```
    

Usage Notes

The name of the model follows the same restrictions as those used for other machine learning models, namely:

  * The schema name, if provided, is limited to 128 characters.
  * The model name is limited to 123 characters and must follow the rules of unquoted identifiers: they contain only alphanumeric characters, the underscore (_), dollar sign ($), and pound sign (#). The initial character must be alphabetic.
  * The model size is limited to 1 gigabyte.
  * The model must not depend on external initializers. To know more about initializers and other ONNX concepts, see <https://onnx.ai/onnx/intro/concepts.md>. 
  * There are default input and output names for input and output attributes for models that are prepared by the Python utility. You can load those models without the JSON parameters. For example: 
```
    EXECUTE DBMS_VECTOR.LOAD_ONNX_MODEL('DM_DUMP', 'my_embedding_model.onnx', 'doc_model'));
```
    




> **note:** See Also:[*Oracle Machine Learning for SQL User’s Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=DMPRG-GUID-A39EB438-4FAE-45C4-9E6B-FBEC7FA84BE8) for examples of using ONNX models for machine learning tasks 

  * [JSON Metadata Parameters for ONNX Models](json-metadata-parameters-onnx-models.md)  
When importing models using the `IMPORT_ONNX_MODEL` (`DBMS_DATA_MINING`), ` LOAD_ONNX_MODEL` (`DBMS_VECTOR`), or `LOAD_ONNX_MODEL_CLOUD` (`DBMS_VECTOR`) procedures, you supply metadata as JSON parameters. 



**Parent topic:** [DBMS_VECTOR](dbms_vector-vecse.md)
