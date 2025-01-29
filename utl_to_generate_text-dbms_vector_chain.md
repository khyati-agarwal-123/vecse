## UTL_TO_GENERATE_TEXT {#GUID-017C9002-194C-48E5-B59B-EF5C60BC8405}

Use the `DBMS_VECTOR_CHAIN.UTL_TO_GENERATE_TEXT` chainable utility function to generate a text response for a given prompt or an image, by accessing third-party text generation models. 

Purpose

To communicate with Large Language Models (LLMs) through natural language conversations. You can generate a textual answer, description, or summary for prompts and images, given as input to LLM-powered chat interfaces.

  * Prompt to Text:

A prompt can be an input text string, such as a question that you ask an LLM. For example, "`What is Oracle Text?`". A prompt can also be a command, such as "`Summarize the following ...`", "`Draft an email asking for ...`", or "`Rewrite the following ...`", and can include results from a search. The LLM responds with a textual answer or description based on the specified task in the prompt. 

For this operation, this API makes a REST call to your chosen remote third-party provider (Cohere, Generative AI, Google AI, Hugging Face, OpenAI, or Vertex AI) or local third-party provider (Ollama).

  * Image to Text:

You can also prompt with a media file, such as an image, to extract text from pictures or photos. You supply a text question as the prompt (such as "`What is this image about?`" or "`How many birds are there in this painting?`") along with the image. The LLM responds with a textual analysis or description of the contents of the image. 

For this operation, this API makes a REST call to your chosen remote third-party provider (Google AI, Hugging Face, OpenAI, or Vertex AI) or local third-party provider (Ollama).




> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

Syntax

This function accepts the input as `CLOB` containing text data (for textual prompts) or as `BLOB` containing media data (for media files such as images). It then processes this information to generate a new `CLOB` containing the generated text. 

  * Prompt to Text:
```
    DBMS_VECTOR_CHAIN.UTL_TO_GENERATE_TEXT (
    DATA          IN CLOB,
    PARAMS        IN JSON default NULL
    ) return CLOB;
```
    

  * Image to Text:
```
    DBMS_VECTOR_CHAIN.UTL_TO_GENERATE_TEXT(
    TEXT_DATA      IN CLOB,
    MEDIA_DATA     IN BLOB,
    MEDIA_TYPE     IN VARCHAR2 default 'image/jpeg',
    PARAMS         IN JSON default NULL
    ) return CLOB;
```
    




DATA and TEXT_DATA

Specify the textual prompt as `CLOB` for the `DATA` or `TEXT_DATA` clause. 

> **note:** Hugging Face uses an image captioning model that does not require a prompt, when giving an image as input. If you input a prompt along with an image, then the prompt will be ignored. 

MEDIA_DATA

Specify the `BLOB` file, such as an image or a visual PDF file. 

MEDIA_TYPE

Specify the image format for the given image or visual PDF file (`BLOB` file) in one of the supported image data MIME types. For example: 

  * For PNG: `image/png`

  * For JPEG: `image/jpeg`

  * For PDF: `application/pdf`




> **note:** For a complete list of the supported image formats, refer to your third-party provider's documentation. 

PARAMS

Specify the following input parameters in JSON format, depending on the service provider that you want to access for text generation:
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
    

**Table: UTL_TO_GENERATE_TEXT Parameter Details**

Parameter | Description  
---|---  
`provider` |  Supported REST provider that you want to access to generate text. <br>Specify one of the following values: <br>For `CLOB` input: <br>* `cohere` <br>`googleai` <br>`huggingface` <br>`ocigenai` <br>`openai` <br>`vertexai`
<br>For `BLOB` input: <br>* `googleai` <br>`huggingface` <br>`openai` <br>`vertexai`  
  
`credential_name` |  Name of the credential in the form: <br> *`schema`*.*`credential_name`* <br>A credential name holds authentication credentials to enable access to your provider for making REST API calls. <br>You need to first set up your credential by calling the `DBMS_VECTOR_CHAIN.CREATE_CREDENTIAL` helper function to create and store a credential, and then refer to the credential name here. See [CREATE_CREDENTIAL](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-A6E28402-DC43-44C6-A1B2-75C3F270DD76).   
`url` |  URL of the third-party provider endpoint for each REST call, as listed in [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09).   
`model` |  Name of the third-party text generation model in the form: <br> *`schema`*.*`model_name`* <br>If the model name is not schema-qualified, then the schema of the procedure invoker is used.<br>**Note**: <br>For Generative AI, all the supported third-party models are listed in [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09).   
`transfer_timeout` |  Maximum time to wait for the request to complete. <br>The default value is `60` seconds. You can increase this value for busy web servers.   
`max_count` |  Maximum number of times the API can be called for a given third-party provider.<br>When set to an integer *n*, `max_count` stops the execution of the API for the given provider beyond *n* times. This prevents accidental calling of a third-party over some limit, for example to avoid surpassing the service amount that you have purchased.   
  
**Additional third-party provider parameters**: 

Optionally, specify additional provider-specific parameters.

**Table: Additional REST Provider Parameter Details**

Parameter | Description  
---|---  
`max_tokens` |  Maximum number of tokens in the output text.  
`temperature` |  Degree of randomness used when generating the output text, in the range of `0.0-5.0`. <br>To generate the same output for a prompt, use `0`. To generate a random new text for that prompt, increase the temperature. <br>**Note**: <br>Start with the temperature set to `0`. If you do not require random results, a recommended temperature value is between `0` and `1`. A higher value is not recommended because a high temperature may produce creative text, which might also include hallucinations.   
`topP` |  Probability of tokens in the output, in the range of `0.0–1.0`. <br>A lower value provides less random responses and a higher value provides more random responses.  
`candidateCount` |  Number of response variations to return, in the range of `1-4`.   
`maxOutputTokens` |  Maximum number of tokens to generate for each response.  
  
Let us look at some example configurations for all third-party providers:

> **note:** Important: 

  * The following examples are for illustration purposes. For accurate and up-to-date information on additional parameters to use, refer to your third-party provider's documentation.

  * For a list of all supported REST endpoint URLs, see [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 




Cohere example:
```
    {
    "provider"       : "cohere",
    "credential_name": "COHERE_CRED",
    "url"            : "https://api.cohere.example.com/chat",
    "model"          : "command"
    }
```
    

Generative AI example:

> **note:** For Generative AI, if you want to pass any additional REST provider-specific parameters, then you must enclose those in `chatRequest`. 
```
    {
    "provider"       : "ocigenai",
    "credential_name": "OCI_CRED",
    "url"            : "https://inference.generativeai.us-example.com/chat",
    "model"          : "cohere.command-r-16k",
    "chatRequest"    : {
    "maxTokens"    : 256
    }
    }
```
    

Google AI example:
```
    {
    "provider"        : "googleai",
    "credential_name" : "GOOGLEAI_CRED",
    "url"             : "https://googleapis.example.com/models/",
    "model"           : "gemini-pro:generateContent"
    }
```
    

Hugging Face example:
```
    {
    "provider"        : "huggingface",
    "credential_name" : "HF_CRED",
    "url"             : "https://api.huggingface.example.com/models/",
    "model"           : "gpt2"
    }
```
    

Ollama example:
```
    {
    "provider"       : "ollama",
    "host"           : "local",
    "url"            : "http://localhost:11434/api/generate",
    "model"          : "phi3:mini"
    }
```
    

OpenAI example:
```
    {
    "provider"        : "openai",
    "credential_name" : "OPENAI_CRED",
    "url"             : "https://api.openai.example.com",
    "model"           : "gpt-4o-mini",
    "max_tokens"      : 60,
    "temperature"     : 1.0
    }
```
    

Vertex AI example:
```
    {
    "provider"         : "vertexai",
    "credential_name"  : "VERTEXAI_CRED",
    "url"              : "https://googleapis.example.com/models/",
    "model"            : "gemini-1.0-pro:generateContent",
    "generation_config": {
    "temperature"    : 0.9,
    "topP"           : 1,
    "candidateCount" : 1,
    "maxOutputTokens": 256
    }
    }
```
    

Examples

  * Prompt to Text:

The following statements generate a text response by making a REST call to Generative AI. The prompt given here is "`What is Oracle Text?`". 

Here, the cohere.command-r-16k and meta.llama-3.1-70b-instruct models are used. You can replace the `model` value with any other supported model that you want to use with Generative AI, as listed in [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 

**Using the cohere.command-r-16k model**: 
```
    -- select example
    
    var params clob;
    exec :params := '
    {
    "provider"       : "ocigenai",
    "credential_name": "OCI_CRED",
    "url"            : "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat",
    "model"          : "cohere.command-r-16k",
    "chatRequest"    : {
    "maxTokens": 256
    }
    }';
    
    select dbms_vector_chain.utl_to_generate_text(
    'What is Oracle Text?',
    json(:params)) from dual;
    
    -- PL/SQL example
    
    declare
    input clob;
    params clob;
    output clob;
    begin
    input := 'What is Oracle Text?';
    
    params := '
    {
    "provider"       : "ocigenai",
    "credential_name": "OCI_CRED",
    "url"            : "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat",
    "model"          : "cohere.command-r-16k",
    "chatRequest"    : {
    "maxTokens": 256
    }
    }';
    
    output := dbms_vector_chain.utl_to_generate_text(input, json(params));
    dbms_output.put_line(output);
    if output is not null then
    dbms_lob.freetemporary(output);
    end if;
    exception
    when OTHERS THEN
    DBMS_OUTPUT.PUT_LINE (SQLERRM);
    DBMS_OUTPUT.PUT_LINE (SQLCODE);
    end;
    /
```
    

**Using the meta.llama-3.1-70b-instruct model**: 
```
    -- select example
    
    var params clob;
    exec :params := '
    {
    "provider"       : "ocigenai",
    "credential_name": "OCI_CRED",
    "url"            : "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat",
    "model"          : "meta.llama-3.1-70b-instruct",
    "chatRequest"    : {
    "topK" : 1
    }
    }';
    
    select dbms_vector_chain.utl_to_generate_text(
    'What is Oracle Text?',
    json(:params)) from dual;
    
    -- PL/SQL example
    
    declare
    input clob;
    params clob;
    output clob;
    begin
    input := 'What is Oracle Text?';
    
    params := '
    {
    "provider"       : "ocigenai",
    "credential_name": "OCI_CRED",
    "url"            : "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat",
    "model"          : "meta.llama-3.1-70b-instruct",
    "chatRequest"    : {
    "topK" : 1
    }
    }';
    
    output := dbms_vector_chain.utl_to_generate_text(input, json(params));
    dbms_output.put_line(output);
    if output is not null then
    dbms_lob.freetemporary(output);
    end if;
    exception
    when OTHERS THEN
    DBMS_OUTPUT.PUT_LINE (SQLERRM);
    DBMS_OUTPUT.PUT_LINE (SQLCODE);
    end;
    /
```
    

**End-to-end examples**: 

To run end-to-end example scenarios, see [Generate Text Response](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-CEE6C9D1-132B-4F0B-ACE0-C2934C326FAE). 

  * Image to Text:

The following statements generate a text response by making a REST call to OpenAI. Here, the input is an image (`sample_image.jpeg`) along with the prompt "`Describe this image?`". 
```
    -- select example
    
    var input clob;
    var media_data blob;
    var media_type clob;
    var params clob;
    
    begin
    :input := 'Describe this image';
    :media_data := load_blob_from_file('DEMO_DIR', 'sample_image.jpeg');
    :media_type := 'image/jpeg';
    :params := '
    {
    "provider"       : "openai",
    "credential_name": "OPENAI_CRED",
    "url"            : "https://api.openai.com/v1/chat/completions",
    "model"          : "gpt-4o-mini",
    "max_tokens"     : 60
    }';
    end;
    /
    
    select dbms_vector_chain.utl_to_generate_text(:input, :media_data, :media_type, json(:params));
    
    -- PL/SQL example
    
    declare
    input clob;
    media_data blob;
    media_type varchar2(32);
    params clob;
    output clob;
    
    begin
    input := 'Describe this image';
    media_data := load_blob_from_file('DEMO_DIR', 'image_file');
    media_type := 'image/jpeg';
    params := '
    {
    "provider"       : "openai",
    "credential_name": "OPENAI_CRED",
    "url"            : "https://api.openai.com/v1/chat/completions",
    "model"          : "gpt-4o-mini",
    "max_tokens"     : 60
    }';
    
    output := dbms_vector_chain.utl_to_generate_text(
    input, media_data, media_type, json(params));
    dbms_output.put_line(output);
    
    if output is not null then
    dbms_lob.freetemporary(output);
    end if;
    if media_data is not null then
    dbms_lob.freetemporary(media_data);
    end if;
    exception
    when OTHERS THEN
    DBMS_OUTPUT.PUT_LINE (SQLERRM);
    DBMS_OUTPUT.PUT_LINE (SQLCODE);
    end;
    /
```
    

**End-to-end examples**: 

To run end-to-end example scenarios, see [Describe Image Content](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-05084961-E744-43DE-A4BC-8375EE3444E6). 




**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
