## About PL/SQL Packages to Generate Embeddings {#GUID-A9FB99E8-68E0-4B2B-9DF2-8C51FB1CA8EA}

Choose to implement Vector Utility PL/SQL packages to perform chunking, embedding, and text generation operations along with text processing and similarity search, both within and outside the database. You can schedule these operations as end-to-end pipelines. The supplied PL/SQL packages for vector generation are `DBMS_VECTOR` and `DBMS_VECTOR_CHAIN`. 

These packages can work with both vector embedding models in ONNX format (by importing these models into the database) and third-party vector embedding models (by calling third-party REST APIs). Each package is made up of subprograms, such as *chainable utility functions* and *vector helper procedures*. 

  * [About Chainable Utility Functions and Common Use Cases](chainable-utility-functions-and-common-use-cases.md)  
These are intended to be a set of chainable and flexible "stages" through which you pass your input data to transform into a different representation, including vectors. 
  * [About Vector Helper Procedures](vector-helper-procedures.md)  
Vector helper procedures let you configure authentication credentials, preferences, and language-specific data for use in chainable utility functions. 
  * [Supplied Vector Utility PL/SQL Packages](supplied-vector-utility-pl-sql-packages.md)  
Use either a lightweight `DBMS_VECTOR` package or a more advanced `DBMS_VECTOR_CHAIN` package with full capabilities. 
  * [Terms of Using Vector Utility PL/SQL Packages](terms-using-vector-utility-pl-sql-packages.md)  
You must understand the terms of using REST APIs that are part of Vector Utility PL/SQL packages. 
  * [Validate JSON Input Parameters](validate-json-input-parameters.md)  
You can optionally validate the structure of your JSON input to the `DBMS_VECTOR.UTL` and `DBMS_VECTOR_CHAIN.UTL` functions, which use JSON to define their input parameters. 



**Related Topics**

  * [Vector Generation Examples](vector-generation-examples.md#GUID-843E4921-A390-41F8-8ED0-91D7B67007B6)



**Parent topic:** [About Vector Generation](vector-generation.md)
