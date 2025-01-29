## UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS {#GUID-C6439E94-4E86-4ECD-954E-4B73D53579DE}

Use the `DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDING` and `DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDINGS` chainable utility functions to generate one or more vector embeddings from textual documents and images. 

Purpose

To automatically generate one or more vector embeddings from textual documents and images.

  * Text to Vector:

You can perform a text-to-embedding transformation by accessing either Oracle Database or a third-party service provider:

    * Oracle Database as the service provider (default setting):

This API calls an ONNX format embedding model that you load into the database.

    * Third-party embedding model:

This API makes a REST API call to your chosen remote service provider (Cohere, Generative AI, Google AI, Hugging Face, OpenAI, or Vertex AI) or local service provider (Ollama).

  * Image to Vector:

You can also perform an image-to-embedding transformation. This API makes a REST call to your chosen image embedding model or multimodal embedding model by Vertex AI. Note that currently Vertex AI is the only supported service provider for this operation.




> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

Syntax

  * Text to Vector:
```
    DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDING (
    DATA           IN CLOB,
    PARAMS         IN JSON default NULL
    ) return VECTOR;
```
```
    DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDINGS (
    DATA           IN VECTOR_ARRAY_T,
    PARAMS         IN JSON default NULL
    ) return VECTOR_ARRAY_T;
```
    

  * Image to Vector:
```
    DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDING (
    DATA           IN BLOB,
    MODALITY       IN VARCHAR2,
    PARAMS         IN JSON default NULL
    ) return VECTOR;
```
    




DATA

  * Text to Vector:

`UTL_TO_EMBEDDING` accepts the input as `CLOB` containing textual data (text strings or small documents). It then converts the text to a single embedding (`VECTOR`). 

`UTL_TO_EMBEDDINGS` converts an array of chunks (`VECTOR_ARRAY_T`) to an array of embeddings (`VECTOR_ARRAY_T`). 

> **note:** Although data is a `CLOB` or a `VECTOR_ARRAY_T` of `CLOB`, the maximum input is 4000 characters. If you have input that is greater, you can use `UTL_TO_CHUNKS` to split the data into smaller chunks before passing in. 

  * Image to Vector:

`UTL_TO_EMBEDDING` accepts the input as `BLOB` containing media data for media files such as images. It then converts the image input to a single embedding (`VECTOR`). 




A generated embedding output includes:
```
    {
    "embed_id"    : NUMBER,
    "embed_data"  : "VARCHAR2(4000)",
    "embed_vector": "CLOB"
    }
```
    

Where, 

  * `embed_id` displays the ID number of each embedding. 

  * `embed_data` displays the input text that is transformed into embeddings. 

  * `embed_vector` displays the generated vector representations. 




MODALITY

For `BLOB` inputs, specify the type of content to vectorize. The only supported value is `image`. 

PARAMS

Specify input parameters in JSON format, depending on the service provider that you want to use.

**If using Oracle Database as the provider**:
```
    {
    "provider" : "database",
    "model"    : ""
    }
```
    

**Table: Database Provider Parameter Details**

Parameter | Description  
---|---  
`provider` |  Specify `database` (default setting) to use Oracle Database as the provider. With this setting, you must load an ONNX format embedding model into the database.   
`model` |  User-specified name under which the imported ONNX embedding model is stored in Oracle Database.<br>If you do not have an embedding model in ONNX format, then perform the steps listed in [Convert Pretrained Models to ONNX Format](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6).   
  
**If using a third-party provider**: 

Set the following parameters along with additional embedding parameters specific to your provider:

  * For `UTL_TO_EMBEDDING`: 
```
    {
    "provider"        : "",
    "credential_name" : "",
    "url"             : "",
    "model"           : "",
    "transfer_timeout": ,
    "max_count": "",
    "": ""
    }
```
    

  * For `UTL_TO_EMBEDDINGS`: 
```
    {
    "provider"        : "",
    "credential_name" : "",
    "url"             : "",
    "model"           : "",
    "transfer_timeout": ,
    "batch_size"      : "",
    "max_count": "",
    "": ""
    }
```
    




**Table: Third-Party Provider Parameter Details**

Parameter | Description  
---|---  
`provider` |  Third-party service provider that you want to access for this operation. A REST call is made to the specified provider to access its embedding model.<br>For image input, specify `vertexai`. <br>For text input, specify one of the following values: <br>* `cohere` <br>`googleai` <br>`huggingface` <br>`ocigenai` <br>`openai` <br>`vertexai`  
  
`credential_name` |  Name of the credential in the form: <br> *`schema`*.*`credential_name`* <br>A credential name holds authentication credentials to enable access to your provider for making REST API calls. <br>You need to first set up your credential by calling the `DBMS_VECTOR_CHAIN.CREATE_CREDENTIAL` helper function to create and store a credential, and then refer to the credential name here. See [CREATE_CREDENTIAL](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-A6E28402-DC43-44C6-A1B2-75C3F270DD76).   
`url` |  URL of the third-party provider endpoint for each REST call, as listed in [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09).   
`model` |  Name of the third-party embedding model in the form: <br> *`schema`*.*`model_name`* <br>If you do not specify a schema, then the schema of the procedure invoker is used.  **Note**: <br>* For Generative AI, all the supported third-party models are listed in [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09).  <br>For accurate results, ensure that the chosen text embedding model matches the vocabulary file used for chunking. If you are not using a vocabulary file, then ensure that the input length is defined within the token limits of your model. <br>To get image embeddings, you can use any image embedding model or multimodal embedding model supported by Vertex AI. Multimodal embedding is a technique that vectorizes data from different modalities such as text and images. <br>When using a multimodal embedding model to generate embeddings, ensure that you use the same model to vectorize both types of content (text and images). By doing so, the resulting embeddings are compatible and situated in the same vector space, which allows for effective comparison between the two modalities during similarity searches.

  
`transfer_timeout` |  Maximum time to wait for the request to complete. <br>The default value is `60` seconds. You can increase this value for busy web servers.   
`batch_size` |  Maximum number of vectors to request at a time. <br>For example, for a batch size of `50`, if `100` chunks are passed, then this API sends two requests with an array of `50` strings each. If `30` chunks are passed (which is lesser than the defined batch size), then the API sends those in a single request. <br>For REST calls, it is more efficient to send a batch of inputs at a time rather than requesting a single input per call. Increasing the batch size can provide better performance, whereas reducing the batch size may reduce memory and data usage, especially if your provider has a rate limit.<br>The default or maximum allowed value depends on the third-party provider settings.  
`max_count` |  Maximum number of times the API can be called for a given third-party provider.<br>When set to an integer *n*, `max_count` stops the execution of the API for the given provider beyond *n* times. This prevents accidental calling of a third-party over some limit, for example to avoid surpassing the service amount that you have purchased.   
  
**Additional third-party provider parameters**: 

Optionally, specify additional provider-specific parameters.

**Table: Additional REST Provider Parameter Details**

Parameter | Description  
---|---  
`input_type` |  Type of input to vectorize.   
  
Let us look at some example configurations for all third-party providers:

> **note:** Important: 

  * The following examples are for illustration purposes. For accurate and up-to-date information on the parameters to use, refer to your third-party provider's documentation.

  * For a list of all supported REST endpoint URLs, see [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 

  * The generated embedding results may be different between requests for the same input and configuration, depending on your embedding model or floating point precision. However, this does not affect your queries (and provides semantically correct results) because the vector distance will be similar.




Cohere example:
```
    {
    "provider"       : "cohere",
    "credential_name": "COHERE_CRED",
    "url"            : "https://api.cohere.example.com/embed",
    "model"          : "embed-english-light-v2.0",
    "input_type"     : "search_query"
    }
```
    

Generative AI example:
```
    {
    "provider"       : "ocigenai",
    "credential_name": "OCI_CRED",
    "url"            : "https://generativeai.oci.example.com/embedText",
    "model"          : "cohere.embed-english-v3.0",
    "batch_size"     : 10
    }
```
    

Google AI example:
```
    {
    "provider"       : "googleai",
    "credential_name": "GOOGLEAI_CRED",
    "url"            : "https://googleapis.example.com/models/",
    "model"          : "embedding-001"
    }
```
    

Hugging Face example:
```
    {
    "provider"       : "huggingface",
    "credential_name": "HF_CRED",
    "url"            : "https://api.huggingface.example.com/",
    "model"          : "sentence-transformers/all-MiniLM-L6-v2"
    }
```
    

Ollama example:
```
    {
    "provider"       : "ollama",
    "host"           : "local",
    "url"            : "http://localhost:11434/api/embeddings",
    "model"          : "phi3:mini"
    }
```
    

OpenAI example:
```
    {
    "provider"       : "openai",
    "credential_name": "OPENAI_CRED",
    "url"            : "https://api.openai.example.com/embeddings",
    "model"          : "text-embedding-3-small"
    }
```
    

Vertex AI example:
```
    {
    "provider"       : "vertexai",
    "credential_name": "VERTEXAI_CRED",
    "url"            : "https://googleapis.example.com/models/",
    "model"          : "textembedding-gecko:predict"
    }
```
    

Examples

You can use `UTL_TO_EMBEDDING` in a `SELECT` clause and `UTL_TO_EMBEDDINGS` in a `FROM` clause, as follows: 

**UTL_TO_EMBEDDING**: 

  * **Text to vector using Generative AI**: 

The following examples use `UTL_TO_EMBEDDING` to generate an embedding with `Hello world` as the input. 

Here, the cohere.embed-english-v3.0 model is used by accessing Generative AI as the provider. You can replace the `model` value with any other supported model that you want to use with Generative AI, as listed in [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 
```
    -- declare embedding parameters
    
    var params clob;
    
    begin
    :params := '
    {
    "provider": "ocigenai",
    "credential_name": "OCI_CRED",
    "url": "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/embedText",
    "model": "cohere.embed-english-v3.0",
    "batch_size": 10
    }';
    end;
    /
    
    -- get text embedding: PL/SQL example
    
    declare
    input clob;
    v vector;
    begin
    input := 'Hello world';
    
    v := dbms_vector_chain.utl_to_embedding(input, json(params));
    dbms_output.put_line(vector_serialize(v));
    exception
    when OTHERS THEN
    DBMS_OUTPUT.PUT_LINE (SQLERRM);
    DBMS_OUTPUT.PUT_LINE (SQLCODE);
    end;
    /
    
    -- get text embedding: select example
    
    select dbms_vector_chain.utl_to_embedding('Hello world', json(:params)) from dual;
```
    

  * **Image to vector using Vertex AI**: 

The following examples use `UTL_TO_EMBEDDING` to generate an embedding by accessing the Vertex AI's multimodal embedding model. 

Here, the input is `parrots.jpg`, `VEC_DUMP` is a local directory that stores the `parrots.jpg` file, and the modality is specified as `image`. 
```
    -- declare embedding parameters
    
    var params clob;
    
    begin
    :params := '
    {
    "provider": "vertexai",
    "credential_name": "VERTEXAI_CRED",
    "url": "https://LOCATION-aiplatform.googleapis.com/v1/projects/PROJECT/locations/LOCATION/publishers/google/models/",
    "model": "multimodalembedding:predict"
    }';
    end;
    /
    
    -- get image embedding: PL/SQL example
    
    declare
    v vector;
    output clob;
    begin
    v := dbms_vector_chain.utl_to_embedding(
    to_blob(bfilename('VEC_DUMP', 'parrots.jpg')), 'image', json(:params));
    output := vector_serialize(v);
    dbms_output.put_line('vector data=' || dbms_lob.substr(output, 100) || '...');
    end;
    /
    
    -- get image embedding: select example
    
    select dbms_vector_chain.utl_to_embedding(
    to_blob(bfilename('VEC_DUMP', 'parrots.jpg')), 'image', json(:params));
```
    

  * **Text to vector using in-database embedding model**: 

The following example uses `UTL_TO_EMBEDDING` to generate a vector embedding by calling an ONNX format embedding model (`doc_model`) loaded into Oracle Database. 

Here, the provider is `database`, and the input is `hello`. 
```
    var params clob;
    exec :params := '{"provider":"database", "model":"doc_model"}';
    
    select dbms_vector_chain.utl_to_embedding('hello', json(:params)) from dual;
```
    

For complete example, see [Convert Text String to Embedding Within Oracle Database](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-1F9E9C73-6118-427E-8FB7-D4EBCCECC6D0). 

  * **End-to-end examples**: 

To run various end-to-end example scenarios using `UTL_TO_EMBEDDING`, see [Generate Embedding](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-813E0E54-9EEF-43FA-A506-1F276D47E7A6). 




**UTL_TO_EMBEDDINGS**: 

  * **Text to vector using in-database embedding model**: 

The following example uses `UTL_TO_EMBEDDINGS` to generate an array of embeddings by calling an ONNX format embedding model (`doc_model`) loaded into Oracle Database. 

Here, the provider is `database`, and the input is a PDF document stored in the `documentation_tab` table. As you can see, you first use `UTL_TO_CHUNKS` to split the data into smaller chunks before passing in to `UTL_TO_EMBEDDINGS`. 
```
    CREATE TABLE doc_chunks as
    (select dt.id doc_id, et.embed_id, et.embed_data, to_vector(et.embed_vector) embed_vector
    from
    documentation_tab dt,
    dbms_vector_chain.utl_to_embeddings(
    dbms_vector_chain.utl_to_chunks(dbms_vector_chain.utl_to_text(dt.data), json('{"normalize":"all"}')),
    json('{"provider":"database", "model":"doc_model"}')) t,
    JSON_TABLE(t.column_value, '$[*]' COLUMNS (embed_id NUMBER PATH '$.embed_id', embed_data VARCHAR2(4000) PATH '$.embed_data', embed_vector CLOB PATH '$.embed_vector')) et
    );
```
    

For complete example, see [SQL Quick Start Using a Vector Embedding Model Uploaded into the Database](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-403EB84E-3047-4905-844C-BD4A8670B8A4). 

  * **End-to-end examples**: 

To run various end-to-end example scenarios using `UTL_TO_EMBEDDINGS`, see [Perform Chunking With Embedding](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-9CB40305-392A-4CB0-AD49-7730504BEC18). 




**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
