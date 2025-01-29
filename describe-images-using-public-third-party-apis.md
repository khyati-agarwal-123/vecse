## Describe Images Using Public REST Providers {#GUID-D386049F-45B4-4047-B1E9-0A5955F9F1BA}

Perform an image-to-text transformation by supplying an image along with a text question as the prompt, using publicly hosted third-party LLMs by Google AI, Hugging Face, OpenAI, or Vertex AI.

Here, you can use the `UTL_TO_GENERATE_TEXT` function from either the `DBMS_VECTOR` or the `DBMS_VECTOR_CHAIN` package, depending on your use case. 

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

To generate a text response by prompting with the following image and the text question "`Describe this image?`", using an external LLM: 

![Description of bird.jpg follows](img/bird.jpg)  


  1. Connect to Oracle Database as a local user.
    1. Log in to SQL*Plus as the `SYS` user, connecting as `SYSDBA`:
```
        conn sys/password as sysdba
```
```
        CREATE TABLESPACE tbs1
        DATAFILE 'tbs5.dbf' SIZE 20G AUTOEXTEND ON
        EXTENT MANAGEMENT LOCAL
        SEGMENT SPACE MANAGEMENT AUTO;
```
```
        SET ECHO ON
        SET FEEDBACK 1
        SET NUMWIDTH 10
        SET LINESIZE 80
        SET TRIMSPOOL ON
        SET TAB OFF
        SET PAGESIZE 10000
        SET LONG 10000
```
        

    2. Create a local user (`docuser`) and grant necessary privileges:
```
        DROP USER docuser cascade;
```
```
        CREATE USER docuser identified by docuser DEFAULT TABLESPACE tbs1 quota unlimited on tbs1;
```
```
        GRANT DB_DEVELOPER_ROLE, create credential to docuser;
```
        

    3. Connect as the local user (`docuser`):
```
        CONN docuser/password
```
        

  2. Set the HTTP proxy server, if configured.
```
    EXEC UTL_HTTP.SET_PROXY(':');
```
    

  3. Grant connect privilege to `docuser` for allowing connection to the host, using the `DBMS_NETWORK_ACL_ADMIN` procedure.

This example uses `*` to allow any host. However, you can explicitly specify the host that you want to connect to. 
```
    BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
    host => '*',
    ace => xs$ace_type(privilege_list => xs$name_list('connect'),
    principal_name => 'docuser',
    principal_type => xs_acl.ptype_db));
    END;
    /
```
    

  4. Create a local directory (`DEMO_DIR`) to store your image file:
```
    create or replace directory DEMO_DIR as '/my_local_dir/';
    
    create or replace function load_blob_from_file(directoryname varchar2, filename varchar2)
    return blob
    is
    filecontent blob := null;
    src_file bfile := bfilename(directoryname, filename);
    offset number := 1;
    begin
    dbms_lob.createtemporary(filecontent, true, dbms_lob.session);
    dbms_lob.fileopen(src_file, dbms_lob.file_readonly);
    dbms_lob.loadblobfromfile(filecontent, src_file,
    dbms_lob.getlength(src_file), offset, offset);
    dbms_lob.fileclose(src_file);
    return filecontent;
    end;
    /
```
    

Upload the image file (for example, `bird.jpg`) to the directory. 

  5. Set up your credentials for the REST provider that you want to access and then call `UTL_TO_GENERATE_TEXT`:

    1. Call `CREATE_CREDENTIAL` to create and store a credential. 

Google AI, Hugging Face, OpenAI, and Vertex AI require the following authentication parameter: 

`{ "access_token": "*``*" }`

You will later refer to this credential name when declaring JSON parameters for the `UTL_TO_GENERATE_TEXT` call. 
```
        exec dbms_vector_chain.drop_credential('');
```
```
        declare
        jo json_object_t;
        begin
        jo := json_object_t();
        jo.put('access_token', '');
        dbms_vector_chain.create_credential(
        credential_name   => '',
        params            => json(jo.to_string));
        end;
        /
```
        

Replace `access_token` and `credential_name` with your own values. For example: 
```
        declare
        jo json_object_t;
        begin
        jo := json_object_t();
        jo.put('access_token', 'AbabA1B123aBc123AbabAb123a1a2ab');
        dbms_vector_chain.create_credential(
        credential_name   => 'HF_CRED',
        params            => json(jo.to_string));
        end;
        /
```
        

    2. Call `UTL_TO_GENERATE_TEXT`: 
```
        -- select example
        
        var input clob;
        var media_data blob;
        var media_type clob;
        var params clob;
        
        begin
        :input := 'Describe this image';
        :media_data := load_blob_from_file('DEMO_DIR', 'bird.jpg');
        :media_type := 'image/jpeg';
        :params := '
        {
        "provider"       : "",
        "credential_name": "",
        "url"            : "",
        "model"          : "",
        "max_tokens"     :
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
        media_data := load_blob_from_file('DEMO_DIR', 'bird.jpg');
        media_type := 'image/jpeg';
        params := '
        {
        "provider"       : "",
        "credential_name": "",
        "url"            : "",
        "model"          : "",
        "max_tokens"     :
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
        

> **note:** For a list of all supported REST endpoint URLs, see [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 

Replace `provider`, `credential_name`, `url`, and `model` with your own values. Optionally, you can specify additional REST provider parameters. This is shown in the following examples: 

Google AI example:
```
        {
        "provider"       : "googleai",
        "credential_name": "GOOGLEAI_CRED",
        "url"            : "https://generativelanguage.googleapis.com/v1beta/models/",
        "model"          : "gemini-pro:generateContent"
        }
```
        

Hugging Face example:
```
        {
        "provider"       : "huggingface",
        "credential_name": "HF_CRED",
        "url"            : "https://api-inference.huggingface.co/models/",
        "model"          : "gpt2"
        }
```
        

> **note:** Hugging Face uses an image captioning model, which does not require a prompt. If you input a prompt along with an image, then the prompt will be ignored. 

OpenAI example:
```
        {
        "provider"       : "openai",
        "credential_name": "OPENAI_CRED",
        "url"            : "https://api.openai.com/v1/chat/completions",
        "model"          : "gpt-4o-mini",
        "max_tokens"     : 60
        }
```
        

Vertex AI example:
```
        {
        "provider"         : "vertexai",
        "credential_name"  : "VERTEXAI_CRED",
        "url"              : "https://LOCATION-aiplatform.googleapis.com/v1/projects/PROJECT/locations/LOCATION/publishers/google/models/",
        "model"            : "gemini-1.0-pro:generateContent",
        "generation_config": {
        "temperature"    : 0.9,
        "topP"           : 1,
        "candidateCount" : 1,
        "maxOutputTokens": 256
        }
        }
```
        

A generated text response to your question may appear as:
```
    This image showcases a stylized, artistic depiction of a hummingbird in
    mid-flight, set against a vibrant red background. The bird is illustrated with a
    mix of striking colors and details - its head and belly are shown in white, with
    a black patterned detailing that resembles stripes or scales. Its long, slender
    beak is depicted in a darker color, extending forwards. The hummingbird's wings
    and tail are rendered in eye-catching shades of orange, purple, and red, with
    texture that suggests a rough, perhaps brush-like stroke.
    
    The background features abstract shapes resembling clouds or wind currents in a white line
    pattern, which adds a sense of motion or air dynamics around the bird. The
    overall use of vivid colors and dynamic patterns gives the image an energetic
    and modern feel.
```
    




This example uses the default settings for each provider. For detailed information on additional parameters, refer to your third-party provider's documentation.

**Related Topics**

  * [UTL_TO_GENERATE_TEXT](utl_to_generate_text-dbms_vector_chain.md#GUID-017C9002-194C-48E5-B59B-EF5C60BC8405)
  * [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12)
  * [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504)



**Parent topic:** [Describe Image Content](describe-image-content.md)
