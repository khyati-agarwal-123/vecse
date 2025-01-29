## DROP_ONNX_MODEL Procedure {#GUID-8031CBC3-3B94-4780-9647-C7D55C0B3331}

This procedure deletes the specified ONNX model.

Syntax
```
    DBMS_VECTOR.DROP_ONNX_MODEL (model_name IN VARCHAR2,
    force      IN BOOLEAN DEFAULT FALSE);
```
    

Parameters

**Table: DROP_ONNX_MODEL Procedure Parameters**

Parameter | Description  
---|---  
`model_name` |  Name of the machine learning ONNX model in the form [*schema_name*.]*model_name*. If you do not specify a schema, then your own schema is used.   
`force` |  Forces the machine learning ONNX model to be dropped even if it is invalid. An ONNX model may be invalid if a serious system error interrupted the model build process.   
  
Usage Note

To drop an ONNX model, you must be the owner or you must have the `DB_DEVELOPER_ROLE`. 

Example

You can use the following command to delete a valid ONNX model named `doc_model` that exists in your schema. 
```
    BEGIN
    DBMS_VECTOR.DROP_ONNX_MODEL(model_name => 'doc_model');
    END;
    /
```
    

**Parent topic:** [DBMS_VECTOR](dbms_vector-vecse.md)
