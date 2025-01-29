## Convert Pretrained Models to ONNX Model: End-to-End Instructions for Text Embedding {#GUID-BB84676B-522B-4667-9B88-0D94740820B6}

This section provides end-to-end instructions from installing the OML4Py client to downloading a pretrained embedding model in ONNX-format using the Python utility package offered by Oracle.

> **note:** This example provides end to end instructions for converting pretrained text models to ONNX models. Steps 1 - 9 are identical for [image models](onnx-pipeline-models-image-embedding.md#GUID-285F1B43-B822-48E2-B29D-4A33C6975140) and [multi-modals](onnx-pipeline-models-multi-modal-embedding.md#GUID-C44EA9C1-2DE0-4E34-AE3D-AFC658DF62AC). You can use appropriate code/syntax mentioned in the corresponding topics to convert image models and multi models to ONNX pipeline models. 

These instructions assume you have configured your Oracle Linux 8 repo in `/etc/yum.repos.d`, configured a Wallet if using an Autonomous Database, and set up a proxy if needed. 

  1. Install Python:
```
    sudo yum install libffi-devel openssl openssl-devel tk-devel xz-devel zlib-devel bzip2-devel readline-devel libuuid-devel ncurses-devel libaio
    mkdir -p $HOME/python
    wget https://www.python.org/ftp/python/3.12.3/Python-3.12.3.tgz
    tar -xvzf Python-3.12.3.tgz --strip-components=1 -C $HOME/python
    cd $HOME/python
    ./configure --prefix=$HOME/python
    make clean; make
    make altinstall
```
    

  2. Set variables `PYTHONHOME`, `PATH`, and ` LD_LIBRARY_PATH`:
```
    export PYTHONHOME=$HOME/python
    export PATH=$PYTHONHOME/bin:$PATH
    export LD_LIBRARY_PATH=$PYTHONHOME/lib:$LD_LIBRARY_PATH
```
    

  3. Create symlink for python3 and pip3: 
```
    cd $HOME/python/bin
    ln -s python3.12 python3
    ln -s pip3.12 pip3
```
    

  4. Install Oracle Instant client if you will be exporting embedded models to the database from Python. If you will be exporting to a file, skip steps 4 and 5 and see the note under environment variables in step 6:
```
    cd $HOME
    wget https://download.oracle.com/otn_software/linux/instantclient/2340000/instantclient-basic-linux.x64-23.4.0.24.05.zip
    unzip instantclient-basic-linux.x64-23.4.0.24.05.zip
```
    

  5. Set variable `LD_LIBRARY_PATH`:
```
    export LD_LIBRARY_PATH=$HOME/instantclient_23_4:$LD_LIBRARY_PATH
```
    

  6. Create an environment file, for example, `env.sh`, that defines the Python and Oracle Instant client environment variables and source these environment variables before each OML4Py client session. Alternatively, add the environment variable definitions to `.bashrc` so they are defined when the user logs into their Linux machine.
```
    # Environment variables for Python
    export PYTHONHOME=$HOME/python
    export PATH=$PYTHONHOME/bin:$PATH
    export LD_LIBRARY_PATH=$PYTHONHOME/lib:$LD_LIBRARY_PATH
```
    

> **note:** Environment variable for Oracle Instant Client - only if the Oracle Instant Client is installed for exporting models to the database. `export LD_LIBRARY_PATH=$HOME/instantclient_23_4:$LD_LIBRARY_PATH`. 

  7. Create a file named requirements.txt that contains the required thid-party packages listed below.
```
    --extra-index-url https://download.pytorch.org/whl/cpu
    pandas==2.1.1
    setuptools==68.0.0
    scipy==1.12.0
    matplotlib==3.8.4
    oracledb==2.2.0
    scikit-learn==1.4.1.post1
    numpy==1.26.4
    onnxruntime==1.17.0
    onnxruntime-extensions==0.10.1
    onnx==1.16.0
    torch==2.2.0+cpu
    transformers==4.38.1
    sentencepiece==0.2.0
```
    

  8. Upgrade pip3 and install the packages listed in requirements.txt.
```
    pip3 install --upgrade pip
    pip3 install -r requirements.txt
```
    

  9. Install OML4Py client. Download OML4Py 2.0.1 client from [OML4Py download page](https://www.oracle.com/database/technologies/oml4py-downloads.md) and upload it to the Linux machine. 
```
    unzip oml4py-client-linux-x86_64-2.0.1.zip
    pip3 install client/oml-2.0-cp312-cp312-linux_x86_64.whl
```
    

  10. Get a list of all preconfigured models. Start Python and import ONNXPipelineConfig from `oml.utils`.
```
    python3
    
    from oml.utils import ONNXPipelineConfig
    
    ONNXPipelineConfig.show_preconfigured()
```
```
    ['sentence-transformers/all-mpnet-base-v2',
    'sentence-transformers/all-MiniLM-L6-v2',
    'sentence-transformers/multi-qa-MiniLM-L6-cos-v1',
    'ProsusAI/finbert',
    'medicalai/ClinicalBERT',
    'sentence-transformers/distiluse-base-multilingual-cased-v2',
    'sentence-transformers/all-MiniLM-L12-v2',
    'BAAI/bge-small-en-v1.5',
    'BAAI/bge-base-en-v1.5',
    'taylorAI/bge-micro-v2',
    'intfloat/e5-small-v2',
    'intfloat/e5-base-v2',
    'prajjwal1/bert-tiny',
    'thenlper/gte-base',
    'thenlper/gte-small',
    'TaylorAI/gte-tiny',
    'infgrad/stella-base-en-v2',
    'sentence-transformers/paraphrase-multilingual-mpnet-base-v2',
    'intfloat/multilingual-e5-base',
    'intfloat/multilingual-e5-small',
    'sentence-transformers/stsb-xlm-r-multilingual',
    'microsoft/codebert-base',
    'Snowflake/snowflake-arctic-embed-xs',
    'Snowflake/snowflake-arctic-embed-s',
    'Snowflake/snowflake-arctic-embed-m',
    'mixedbread-ai/mxbai-embed-large-v1',
    'openai/clip-vit-base-patch32',
    'google/vit-base-patch16-224',
    'microsoft/resnet-18',
    'microsoft/resnet-50',
    'WinKawaks/vit-tiny-patch16-224',
    'Falconsai/nsfw_image_detection',
    'WinKawaks/vit-small-patch16-224',
    'nateraw/vit-age-classifier',
    'rizvandwiki/gender-classification',
    'AdamCodd/vit-base-nsfw-detector',
    'trpakov/vit-face-expression',
    'BAAI/bge-reranker-base']
```
    

  11. Choose from: 
     * To generate an ONNX file that you can manually upload to the database using the `DBMS_VECTOR.LOAD_ONNX_MODEL`, refer to step 3 of [SQL Quick Start](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-403EB84E-3047-4905-844C-BD4A8670B8A4) and skip steps 12 and 13. 
     * To upload the model directly into the database, skip this step and proceed to step 12.

Export a preconfigured embedding model to a local file. Import ONNXPipeline and ONNXPipelineConfig from `oml.utils`. This exports the ONNX-format model to your local file system.
```
    from oml.utils import ONNXPipeline, ONNXPipelineConfig
    
    # Export to file
    pipeline = ONNXPipeline(model_name="sentence-transformers/all-MiniLM-L6-v2")
    pipeline.export2file("your_preconfig_file_name",output_dir=".")
```
    

Move the ONNX file to a directory on the database server, and create a directory on the file system and in the database for the import. 
```
    mkdir -p /tmp/models
    sqlplus / as sysdba
    alter session set container=;
```
    

Apply the necessary permissions and grants. 
```
    -- directory to store ONNX files for import
    CREATE DIRECTORY ONNX_IMPORT AS '/tmp/models';
    -- grant your OML user read and write permissions on the directory
    GRANT READ, WRITE ON DIRECTORY ONNX_IMPORT to OMLUSER;
    -- grant to allow user to import the model
    GRANT CREATE MINING MODEL TO OMLUSER;
```
    

Use the `DBMS_VECTOR.LOAD_ONNX_MODEL` procedure to load the model in your OML user schema. In this example, the procedure loads the ONNX model file named `all-MiniLM-L6.onnx` from the ONNX_IMPORT directory into the database as a model named ALL_MINILM_L6. 
```
    BEGIN
    DBMS_VECTOR.LOAD_ONNX_MODEL(
    directory => 'ONNX_IMPORT',
    file_name => 'all-MiniLM-L6-v2.onnx',
    model_name => 'ALL_MINILM_L6',
    metadata => JSON('{"function" : "embedding", "embeddingOutput" : "embedding", "input": {"input": ["DATA"]}}'));
    END;
```
    

  12. Export a preconfigured embedding model to the database. If using a database connection to update to match your credentials and database environment.

> **note:** To ensure step 12 works properly, complete steps 4 and 5 first. 
```
    # Import oml library and EmbeddingModel from oml.utils
    import oml
    from oml.utils import ONNXPipeline, ONNXPipelineConfig
    
    # Set embedded mode to false for Oracle Database on premises. This is not supported or required for Oracle Autonomous Database.
    oml.core.methods.__embed__ = False
    
    # Create a database connection.
    
    # Oracle Database on-premises
    oml.connect("", "", port= host="", service_name="")
    
    # Oracle Autonomous Database
    oml.connect(user="", password="", dsn="myadb_low")
    pipeline = ONNXPipeline(model_name="sentence-transformers/all-MiniLM-L6-v2")
    em.export2db("ALL_MINILM_L6")
```
    

Query the model and its views, and you can generate embeddings from Python or SQL.
```
    import oracledb
    cr = oml.cursor()
    data = cr.execute("select vector_embedding(ALL_MINILM_L6 using 'RES' as DATA)AS embedding from dual")
    data.fetchall()
```
```
    SELECT VECTOR_EMBEDDING(ALL_MINILM_L6 USING 'RES' as DATA) AS embedding;
```
    

  13. Verify the model exists using SQL:
```
    sqlplus $USER/pass@PDBNAME;
```
```
    select model_name, algorithm, mining_function from user_mining_models where  model_name='ALL_MINILM_L6';
```
```
    ---------------------------------------------------------------------------
    MODEL_NAME                 ALGORITHM                      MINING_FUNCTION
    ------------------------------ -------------------------------------------
    ALL_MINILM_L6              ONNX                           EMBEDDING
```
    




**Parent topic:** [Import Pretrained Models in ONNX Format](import-pretrained-models-onnx-format-vector-generation-database.md)
