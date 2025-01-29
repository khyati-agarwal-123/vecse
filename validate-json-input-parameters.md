## Validate JSON Input Parameters {#GUID-BD911C3C-D17F-4A8D-90BF-5A82A604D7C8}

You can optionally validate the structure of your JSON input to the `DBMS_VECTOR.UTL` and `DBMS_VECTOR_CHAIN.UTL` functions, which use JSON to define their input parameters. 

The JSON data is schemaless, so the amount of validation that Vector Utility package APIs do at runtime is minimal for better performance. The APIs validate only the mandatory JSON parameters, that is, the parameters that you supply for the APIs to run (not optional JSON parameters and attributes).

Before calling an API, you can use subprograms in the `DBMS_JSON_SCHEMA` package to test whether the input data to be specified in the `PARAMS` clause is valid with respect to a given JSON schema. This offers more flexibility and also ensures that only schema-valid data is inserted in a JSON column. 

Validate JSON input parameters for the `DBMS_VECTOR.UTL` and `DBMS_VECTOR_CHAIN.UTL` functions against the following schema: 

  * **For the Database Provider**: 

    * `SCHEMA_CHUNK`
```
        { "title" : "utl_to_chunks",
        "description" : "Chunk parameters",
        "type" : "object",
        "properties" : {
        "by"           : {"type" : "string", "enum" : [ "chars", "characters", "words", "vocabulary" ]   },
        "max"          : {"type" : "string", "pattern" : "^[1-9][0-9]*$" },
        "overlap"      : {"type" : "string", "pattern" : "^[0-9]+$" },
        "split"        : {"type" : "string", "enum" : [ "none", "newline", "blankline", "space", "recursively", "custom" ]  },
        "vocabulary"   : {"type" : "string"  },
        "language"     : {"type" : "string"  },
        "normalize"    : {"type" : "string", "enum" : [ "all", "none", "options" ]  },
        "norm_options" : {"type" : "array",  "items": {  { "type": "string", "enum": ["widechar", "whitespace", "punctuation"] }  },
        "custom_list"  : {"type" : "array",  "items": { "type": "string" }  },
        "extended"     : {"type" : "boolean" } },
        "additionalProperties" : false
        }
```
        

    * `SCHEMA_VOCAB`
```
        { "title" : "create_vocabulary",
        "description" : "Create vocabulary parameters",
        "type" : "object",
        "properties" : {
        "table_name"      : {"type" : "string" },
        "column_name"     : {"type" : "string" },
        "vocabulary_name" : {"type" : "string" },
        "format"          : {"type" : "string", "enum" : [ "BERT", "GPT2", "XLM" ]  },
        "cased"           : {"type" : "boolean" } },
        "additionalProperties" : false,
        "required" : [ "table_name", "column_name", "vocabulary_name" ]
        }
```
        

    * `SCHEMA_LANG`
```
        { "title" : "create_lang_data",
        "description" : "Create language data parameters",
        "type" : "object",
        "properties" : {
        "table_name"      : {"type" : "string" },
        "column_name"     : {"type" : "string" },
        "preference_name" : {"type" : "string" },
        "language"        : {"type" : "string" } },
        "additionalProperties" : false,
        "required" : [ "table_name", "column_name", "preference_name", "language" ]
        }
```
        

    * `SCHEMA_TEXT`
```
        { "title": "utl_to_text",
        "description": "To text parameters",
        "type" : "object",
        "properties" : {
        "plaintext"       : {"type" : "boolean" },
        "charset"         : {"type" : "string", "enum" : [ "UTF8" ] } },
        "additionalProperties": false
        }
```
        

    * `SCHEMA_DBEMB`
```
        { "title" : "utl_to_embedding",
        "description" : "To DB embeddings parameters",
        "type" : "object",
        "properties" : {
        "provider"  : {"type" : "string" },
        "model"     : {"type" : "string" } },
        "additionalProperties": true,
        "required" : [ "provider", "model" ]
        }
```
        

    * `SCHEMA_SUM`
```
        { "title" : "utl_to_summary",
        "description" : "To summary parameters",
        "type" : "object",
        "properties" : {
        "provider"       : {"type" : "string" },
        "numParagraphs"  : {"type" : "number" },
        "language"       : {"type" : "string" },
        "glevel"         : {"type" : "string" }
        },
        "additionalProperties" : true,
        "required" : [ "provider" ]
        }
```
        

  * **For REST Providers**: 

`SCHEMA_REST`
```
    {  "title" : "REST parameters",
    "description" : "REST versions of utl_to_embedding, utl_to_summary, utl_to_generate_text",
    "type" : "object",
    "properties" : {
    "provider"         : {"type" : "string" },
    "credential_name"  : {"type" : "string" },
    "url"              : {"type" : "string" },
    "model"            : {"type" : "string" }
    },
    "additionalProperties" : true,
    "required" : [ "provider", "credential_name", "url", "model" ] }
```
    

Note that all the REST calls to third-party service providers share the same schema for their respective embedding, summarization, and text generation operations.




**Examples**: 

  * To validate your JSON data against JSON schema, use the PL/SQL function or procedure `DBMS_JSON_SCHEMA.is_valid()`. 

The function returns `1` for valid and `0` for invalid (invalid data can optionally raise an error). The procedure returns `TRUE` for valid and `FALSE` for invalid as the value of an `OUT` parameter. 
```
    l_valid := sys.DBMS_JSON_SCHEMA.is_valid(params, json(SCHEMA), dbms_json_schema.RAISE_ERROR);
```
    

  * If you use the procedure (not function) `is_valid`, then you have access to the validation errors report as an `OUT` parameter. This use of the procedure checks data against schema, providing output in parameters validity (`BOOLEAN`) and errors (`JSON`). 
```
    sys.DBMS_JSON_SCHEMA.is_valid(params, json(SCHEMA), l_valid, l_errors);
```
    

  * To read a detailed validation report of errors, use the PL/SQL procedure `DBMS_JSON_SCHEMA.validate_report`. 

If you use the function (not the procedure) `is_valid`, then you do not have access to such a report. Instead of using the function `is_valid`, you can use the PL/SQL function `DBMS_JSON_SCHEMA.validate_report` in a SQL query to validate and return the same full validation information that the reporting `OUT` parameter of the procedure `is_valid` provides, as a JSON type instance. 
```
    SELECT JSON_SERIALIZE(DBMS_JSON_SCHEMA.validate_report('json',SCHEMA) returning varchar2 PRETTY);
```
    




**Related Topics**

  * [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12)
  * [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504)
  * [DBMS_JSON_SCHEMA](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-63089EEE-6DF8-41C9-9871-11B94D7551B4)



**Parent topic:** [About PL/SQL Packages to Generate Embeddings](pl-sql-packages-generate-embeddings.md)
