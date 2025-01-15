##  Convert Pretrained Models to ONNX Format {#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6} 

OML4Py enables the use of text transformers from Hugging Face by converting them into ONNX format models. OML4Py also adds the necessary tokenization and post-processing. The resulting ONNX pipeline is then imported into the database and can be used to generate embeddings for AI Vector Search. 

> **note:** 

  * This feature will  only  work on OML4Py client. It is not supported on the OML4Py server. 

  * In-database embedding models must include tokenization and post-processing. Providing only the core ONNX model to ` DBMS_VECTOR.LOAD_ONNX_MODEL ` is insufficient because you need to handle tokenization externally, pass tensors into the SQL operator, and convert output tensors into vectors. 

  * Oracle is providing a [ Hugging Face all-MiniLM-L12-v2 model ](https://adwc4pm.objectstorage.us-ashburn-1.oci.customer-oci.com/p/VBRD9P8ZFWkKvnfhrWxkpPe8K03-JIoM5h_8EJyJcpE80c108fuUjg7R5L5O7mMZ/n/adwc4pm/b/OML-Resources/o/all_MiniLM_L12_v2_augmented.zip) in ONNX format, available to download directly to the database using ` DBMS_VECTOR.LOAD_ONNX_MODEL ` . For more information, see the blog post [ Now Available! Pre-built Embedding Generation model for Oracle Database 23ai ](https://blogs.oracle.com/machinelearning/post/use-our-prebuilt-onnx-model-now-available-for-embedding-generation-in-oracle-database-23ai) . 

If you do not have a pretrained embedding model in ONNX-format to generate embeddings for your data, Oracle offers a Python utility package that downloads pretrained models from an external source, converts the model to ONNX format augmented with pre-processing and post-processing steps, and imports the resulting ONNX-format model into Oracle Database. Use the ` DBMS_VECTOR.LOAD_ONNX_MODEL ` procedure or OML4Py's ` export2db() ` function to import the file as a mining model.. Then leverage the in-database ONNX Runtime with the ONNX model to produce vector embeddings. 

At a high level, the Python utility package performs the following tasks: 
    * Downloads the pretrained model from external source to your system 
    * Augments the model with pre-processing and post-processing steps and creates a new ONNX model 
    * Validates the augmented ONNX model 
    * Loads into the database as a mining model or optionally exports to a file 

The Python utility can take any of the models in the preconfigured list as input. Alternatively, you can use the built-in template that contains common configurations for certain groups of models such as text-based models. To understand what a preconfigured list, what is a built-in template is, and how to use them, read further. 

Limitations 

This table describes the limitations of the Python utility package. 

> **note:** This feature is available with the OML4Py client only. 

Parameter  |  Description   
---|---  
` Transformer Model Type ` |  Currently supported only for text transformers.   
` Model Size ` |  Model size should be less than 1GB. Quantization can help reduce the size.   
` Tokenizers ` |  Must be either ` BERT ` , ` GPT2 ` , ` SENTENCEPIECE ` , or ` ROBERTA ` .   
  
Preconfigured List of Models 

Preconfigured list of models are common models from external resource repositories that are provided with the Python utility. The preconfigured models have an existing specification. Users can create their own specification using the text template as a starting point. To get a list of all model names in the preconfigured list, you can use the ` show_preconfigured ` function. 

Templates 

The Python utility package provides built-in text template for you to configure the pretrained models with pre-processing and post-processing operations. The template has a default specification for the pretrained models. This specification can be changed or augmented to create custom configurations. The text template uses Mean Pooling and Normalization as post-processing operations by default. 

The Python utility package provides the following classes: 
      * ` EmbeddingModelConfig `

      * ` EmbeddingModel `

To learn more about the Python classes, their properties, and to configure the properties, see [ Python Classes to Convert Pretrained Models to ONNX Models ](python-classes-convert-pretrained-models-onnx-models.html#GUID-53F409DB-CC9A-406E-8697-0A824A415914) . 

To use the Python utility, ensure that you have the following: 
        * OML4Py Client running on Linux X64 for On-Premises Databases 

        * Python 3.12 (the earlier versions are not compatible) 

. 

          1. Start Python in your work directory. 
                    
                                        ```
                    python3
                    ```
                    
                                        ```
                    Python 3.12.3 | (main, Feb 27 2024, 17:35:02) [GCC 11.2.0] on linux
                    Type "help", "copyright", "credits" or "license" for more information.
                    ```

          2. On the OML4Py client, load the Python classes: 
                    
                                        ```
                    from oml.utils import EmbeddingModel, EmbeddingModelConfig
                    ```

          3. You can get a list of all preconfigured models by running the following: 
                    
                                        ```
                    EmbeddingModelConfig.show_preconfigured()
                    ```

          4. To get a list of available templates: 
                    
                                        ```
                    EmbeddingModelConfig.show_templates()
                    ```

          5. Choose from: 
             * Generate an ONNX file from the preconfigureded model "sentence-transformers/all-MiniLM-L6-v2": 
                        
                                                ```
                        
                        #generate from preconfigureded model "sentence-transformers/all-MiniLM-L6-v2"
                        em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2")
                        em.export2file("your_preconfig_file_name",output_dir=".")
                        ```

             * Generate an ONNX model from the preconfigured model "sentence-transformers/all-MiniLM-L6-v2" in the database: 
                        
                                                ```
                        
                        #generate from preconfigureded model "sentence-transformers/all-MiniLM-L6-v2"
                        em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2")
                        em.export2db("your_preconfig_model_name")
                        ```

             * Generate an ONNX file using the provided text template: 
                        
                                                ```
                        #generate using the "text" template
                        config = EmbeddingModelConfig.from_template("text",max_seq_length=512)
                        em = EmbeddingModel(model_name="intfloat/e5-small-v2",config=config)
                        em.export2file("your_template_file_name",output_dir=".")
                        ```

Let's understand the code: 

` from oml.utils import EmbeddingModel, EmbeddingModelConfig `

This line imports two classes, ` EmbeddingModel ` and ` EmbeddingModelConfig ` . 

In the preconfigured models first example: 
               * ` em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2") ` creates an instance of the ` EmbeddingModel ` class, loading a pretrained model specified by the ` model_name ` parameter. ` em ` is the embedding model object. ` sentence-transformers/all-MiniLM-L6-v2 ` is the model name for computing sentence embeddings. This is the model name under Hugging Face. Oracle supports models from Hugging Face. 

               * The ` export2file ` command creates an ONNX format model with a user-specified model name in the file. ` your_preconfig_file_name ` is a user defined ONNX model file name. 

               * ` output_dir="." ` specifies the output directory where the file will be saved. The ` "." ` denotes the current directory (that is, the directory from which the script is running). 

In the preconfigured models second example: 
                 * ` em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2") ` creates an instance of the ` EmbeddingModel ` class, loading a pretrained model specified by the ` model_name ` parameter. ` em ` is the embedding model object. ` sentence-transformers/all-MiniLM-L6-v2 ` is the model name for computing sentence embeddings. This is the model name under Hugging Face. Oracle supports models from Hugging Face. 

                 * The ` export2db ` command creates an ONNX format model with a user defined model name in the database. ` your_preconfig_model_name ` is a user defined ONNX model name. 

In the template example: 
                   * ` config = EmbeddingModelConfig.from_template("text", max_seq_length=512) ` : This line creates a configuration object for an embedding model using a method called ` from_template ` . The ` "text" ` argument indicates the name of the template. The ` max_seq_length=512 ` parameter specifies the maximum length of input to the model as number of tokens. There is no default value. Specify this value for models that are not preconfigured. 

                   * ` em = EmbeddingModel(model_name="intfloat/e5-small-v2", config=config) ` initializes an ` EmbeddingModel ` instance with a specific model and the previously defined configuration. The ` model_name="intfloat/e5-small-v2" ` argument specifies the name or identifier of the pretrained model to be loaded. 

                   * The ` export2file ` command creates an ONNX format model with a user defined model name in the file. ` your_template_file_name ` is a user defined ONNX model name. 

                   * ` output_dir="." ` specifies the output directory where the file will be saved. The ` "." ` denotes the current directory (that is, the directory from which the script is running). 

> **note:** 
                     * The model size is limited to 1 gigabyte. For models larger than 400MB, Oracle recommends quantization. 

**Quantization** reduces the model size by converting model weights from high-precision representation to low-precision format. The quantization option converts the weights to INT8. The smaller model size enables you to cache the model in shared memory further improving the performance. 

                     * The ` .onnx ` file is created with opset version 17 and ir version 8. For more information about these version numbers, see [ https://onnxruntime.ai/docs/reference/compatibility.md#onnx-opset-support ](https://onnxruntime.ai/docs/reference/compatibility.md#onnx-opset-support) . 

                     * Choose from: 
                       * Load the ONNX model to your database using PL/SQL: 
                         1. Move the ONNX file named ` your_template_file_name ` to a directory on the database server, and create a directory on the file system and in the database for the import. 
                                                
                                                                                                ```
                                                mkdir -p /tmp/models
                                                sqlplus / as sysdba
                                                alter session set container=ORCLPDB;
                                                ```

Apply the necessary permissions and grants. In this example, we are using a pluggable database named ORCLPDB. 
                                                
                                                                                                ```
                                                -- directory to store ONNX files for import
                                                CREATE DIRECTORY ONNX_IMPORT AS '/tmp/models';
                                                -- grant your OML user read and write permissions on the directory
                                                GRANT READ, WRITE ON DIRECTORY ONNX_IMPORT to OMLUSER;
                                                -- grant to allow user to import the model
                                                GRANT CREATE MINING MODEL TO OMLUSER;
                                                ```

                         2. Load the ONNX model to your database using Python: 
                         3. Exit Python. 
                                                
                                                                                                ```
                                                exit()
                                                ```

                         4. Inspect if the converted models are present in your directory. 

> **note:** ONNX files are only created when ` export2file ` is used. If ` export2db ` is used, no local ONNX files will be generated and the model will be saved in the database. 

List all files in the current directory that have the extension ` onnx ` . 
                                                
                                                                                                ```
                                                ls -ltr *.onnx
                                                ```

The Python utility package validates the embedding text model before you can run them using ONNX Runtime. Oracle supports ONNX embedding models that conform to ` string ` as input and ` float32 [  ]  ` as output. 

If the input or output of the model doesn't conform to the description, you receive an error during the import. 

                         5. Verify the model is in the database: 

At your operating system prompt, start SQL*Plus, connect to it : 
                                                
                                                                                                ```
                                                sqlplus OML_USER/password@pdbname_medium;
                                                ```
                                                
                                                                                                ```
                                                
                                                SQL*Plus: Release 23.0.0.0.0 - Production on Wed May 1 15:33:29 2024
                                                Version 23.4.0.24.05
                                                
                                                Copyright (c) 1982, 2024, Oracle. All rights reserved.
                                                
                                                Last Successful login time: Wed May 01 2024 15:27:06 -07:00
                                                
                                                Connected to:
                                                Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - Production
                                                Version 23.4.0.24.05
                                                ```

List the ONNX model in the database to make sure it was loaded: 
                                                
                                                                                                ```
                                                SQL> select MODEL_NAME, ALGORITHM from user_mining_models WHERE MODEL_NAME='YOUR_PRECONFIG_MODEL_NAME';
                                                ```
                                                
                                                                                                ```
                                                MODEL_NAME ALGORITHM
                                                -------------------------------------
                                                YOUR_PRECONFIG_MODEL_NAME ONNX
                                                ```

` DBMS_VECTOR.LOAD_ONNX_MODEL ` is only needed if ` export2file ` was used to save the ONNX model file to the local system instead of using ` export2db ` to save the model in the database. The ` DBMS_VECTOR.LOAD_ONNX_MODEL ` imports the ONNX format model into the Oracle Database to leverage the in-database ONNX Runtime to produce vector embeddings using the ` VECTOR_EMBEDDING ` SQL operator. 

> **note:** See Also: 
                            * [ *Oracle Database SQL Language Reference*  ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SQLRF-GUID-5ED78260-6D21-4B6B-86E0-A1E70EFA11CA) for information about the ` VECTOR_EMBEDDING ` SQL function 
                            * [ *Oracle Database PL/SQL Packages and Types Reference*  ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-BBAAE33A-412B-455A-8FE0-81BE31120300) for information about the ` IMPORT_ONNX_MODEL ` procedure 
                            * [ *Oracle Database PL/SQL Packages and Types Reference*  ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-7F1D7992-D8F7-4AD9-9BF6-6EFFC1B0617A) for information about the ` LOAD_ONNX_MODEL ` procedure 
                            * [ *Oracle Machine Learning for SQL Concepts*  ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=DMCON-GUID-6AEA7A0E-78E0-4083-A126-4516EB98175A) for more information about importing pretrained embedding models in ONNX format and generating vector embeddings 
                            * [ https://onnx.ai/onnx/intro/ ](https://onnx.ai/onnx/intro/) for ONNX documentation 

                              * [ Convert Pretrained Models to ONNX Model: End-to-End Instructions ](convert-pretrained-models-onnx-model-end-end-instructions.html)   
This section provides end-to-end instructions from installing the OML4Py client to downloading a pretrained embedding model in ONNX-format using the Python utility package offered by Oracle. 
                              * [ Python Classes to Convert Pretrained Models to ONNX Models ](python-classes-convert-pretrained-models-onnx-models.html)   
Explore the functions and attributes of the ` EmbeddingModelConfig ` class and ` EmbeddingModel ` class within Python. These classes are designed to configure pretrained embedding models. 

**Parent topic:** [ Import Pretrained Models in ONNX Format for Vector Generation Within the Database ](import-pretrained-models-onnx-format-vector-generation-database.html)