## Convert Text String to Embedding Using Public REST Providers {#GUID-C693ED29-A963-4F19-BF2B-028545A3AA00}

Perform a text-to-embedding transformation, using publicly hosted third-party embedding models by Cohere, Generative AI, Google AI, Hugging Face, OpenAI, or Vertex AI.

You can use third-party embedding models to vectorize your data that is used to populate a vector index. Note that you must use the same embedding model on both the data to be indexed and the user's input query. In this example, you can see how to vectorize a user's input query on the fly.

Here, you can call the chainable utility function `UTL_TO_EMBEDDING` (note the singular "embedding") from either the `DBMS_VECTOR` or the `DBMS_VECTOR_CHAIN` package, depending on your use case. This example uses the `DBMS_VECTOR.UTL_TO_EMBEDDING` API. 

`UTL_TO_EMBEDDING` directly returns a `VECTOR` type (not an array). 

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

To convert a user's input text "`hello`" to a vector embedding, using a public third-party embedding model: 

  1. Connect to Oracle Database as a local user.
    1. Log in to SQL*Plus as the `sys` user, connecting as `sysdba`:
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
    

  4. Set up your credentials for the REST provider that you want to access and then call `UTL_TO_EMBEDDING`.

     * **Using Generative AI**: 

       1. Run `DBMS_VECTOR.CREATE_CREDENTIAL` to create and store an OCI credential (`OCI_CRED`). 

Generative AI requires the following authentication parameters:
```
            {
            "user_ocid"       : "",
            "tenancy_ocid"    : "",
            "compartment_ocid": "",
            "private_key"     : "",
            "fingerprint"     : ""
            }
```
            

You will later refer to this credential name when declaring JSON parameters for the `UTL_to_EMBEDDING` call. 

> **note:** 

The generated private key may appear as:
```
            -----BEGIN RSA PRIVATE KEY-----
            
            -----END RSA PRIVATE KEY-----
```
            

You pass the *``*  value (excluding the `BEGIN` and `END` lines), either as a single line or as multiple lines. 
```
            exec dbms_vector.drop_credential('OCI_CRED');
```
```
            declare
            jo json_object_t;
            begin
            jo := json_object_t();
            jo.put('user_ocid','');
            jo.put('tenancy_ocid','');
            jo.put('compartment_ocid','');
            jo.put('private_key','');
            jo.put('fingerprint','');
            dbms_vector.create_credential(
            credential_name   => 'OCI_CRED',
            params            => json(jo.to_string));
            end;
            /
```
            

Replace all the authentication parameter values. For example:
```
            declare
            jo json_object_t;
            begin
            jo := json_object_t();
            jo.put('user_ocid','ocid1.user.oc1..aabbalbbaa1112233aabbaabb1111222aa1111bb');
            jo.put('tenancy_ocid','ocid1.tenancy.oc1..aaaaalbbbb1112233aaaabbaa1111222aaa111a');
            jo.put('compartment_ocid','ocid1.compartment.oc1..ababalabab1112233abababab1111222aba11ab');
            jo.put('private_key','AAAaaaBBB11112222333...AAA111AAABBB222aaa1a/+');
            jo.put('fingerprint','01:1a:a1:aa:12:a1:12:1a:ab:12:01:ab:a1:12:ab:1a');
            dbms_vector.create_credential(
            credential_name   => 'OCI_CRED',
            parameters        => json(jo.to_string));
            end;
            /
```
            

       2. Call `DBMS_VECTOR.UTL_TO_EMBEDDING`: 

Here, the cohere.embed-english-v3.0 model is used. You can replace `model` with your own value, as required. 

> **note:** For a list of all REST endpoint URLs and models that are supported to use with Generative AI, see [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 
```
            -- select example
            
            var params clob;
            exec :params := '
            {
            "provider": "ocigenai",
            "credential_name": "OCI_CRED",
            "url": "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/embedText",
            "model": "cohere.embed-english-v3.0",
            "batch_size": 10
            }';
            
            select dbms_vector.utl_to_embedding('hello', json(:params)) from dual;
            
            -- PL/SQL example
            
            declare
            input clob;
            params clob;
            v vector;
            begin
            input := 'hello';
            params := '
            {
            "provider": "ocigenai",
            "credential_name": "OCI_CRED",
            "url": "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/embedText",
            "model": "cohere.embed-english-v3.0",
            "batch_size": 10
            }';
            
            v := dbms_vector.utl_to_embedding(input, json(params));
            dbms_output.put_line(vector_serialize(v));
            exception
            when OTHERS THEN
            DBMS_OUTPUT.PUT_LINE (SQLERRM);
            DBMS_OUTPUT.PUT_LINE (SQLCODE);
            end;
            /
```
            

     * **Using Cohere, Google AI, Hugging Face, OpenAI, and Vertex AI**: 

       1. Run `DBMS_VECTOR.CREATE_CREDENTIAL` to create and store a credential. 

Cohere, Google AI, Hugging Face, OpenAI, and Vertex AI require the following authentication parameter: 

`{ "access_token": "*``*" }`

You will later refer to this credential name when declaring JSON parameters for the `UTL_to_EMBEDDING` call. 
```
            exec dbms_vector.drop_credential('');
```
```
            declare
            jo json_object_t;
            begin
            jo := json_object_t();
            jo.put('access_token', '');
            dbms_vector.create_credential(
            credential_name   => '',
            params            => json(jo.to_string));
            end;
            /
```
            

Replace the `access_token` and `credential_name` values. For example: 
```
            declare
            jo json_object_t;
            begin
            jo := json_object_t();
            jo.put('access_token', 'AbabA1B123aBc123AbabAb123a1a2ab');
            dbms_vector.create_credential(
            credential_name   => 'HF_CRED',
            params            => json(jo.to_string));
            end;
            /
```
            

       2. Call `DBMS_VECTOR.UTL_TO_EMBEDDING`: 
```
            -- select example
            
            var params clob;
            exec :params := '
            {
            "provider": "",
            "credential_name": "",
            "url": "",
            "model": ""
            }';
            
            select dbms_vector.utl_to_embedding('hello', json(:params)) from dual;
            
            -- PL/SQL example
            
            declare
            input clob;
            params clob;
            v vector;
            begin
            input := 'hello';
            
            params := '
            {
            "provider": "",
            "credential_name": "",
            "url": "",
            "model": ""
            }';
            
            v := dbms_vector.utl_to_embedding(input, json(params));
            dbms_output.put_line(vector_serialize(v));
            exception
            when OTHERS THEN
            DBMS_OUTPUT.PUT_LINE (SQLERRM);
            DBMS_OUTPUT.PUT_LINE (SQLCODE);
            end;
            /
```
            

> **note:** For a complete list of all supported REST endpoint URLs, see [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 

Replace `provider`, `credential_name`, `url`, and `model` with your own values. Optionally, you can specify additional REST provider parameters. This is shown in the following examples: 

Cohere example:
```
            {
            "provider"       : "cohere",
            "credential_name": "COHERE_CRED",
            "url"            : "https://api.cohere.ai/v1/embed",
            "model"          : "embed-english-light-v2.0",
            "input_type"     : "search_query"
            }
```
            

Google AI example:
```
            {
            "provider"       : "googleai",
            "credential_name": "GOOGLEAI_CRED",
            "url"            : "https://generativelanguage.googleapis.com/v1beta/models/",
            "model"          : "embedding-001"
            }
```
            

Hugging Face example:
```
            {
            "provider"       : "huggingface",
            "credential_name": "HF_CRED",
            "url"            : "https://api-inference.huggingface.co/pipeline/feature-extraction/",
            "model"          : "sentence-transformers/all-MiniLM-L6-v2"
            }
```
            

OpenAI example:
```
            {
            "provider"       : "openai",
            "credential_name": "OPENAI_CRED",
            "url"            : "https://api.openai.com/v1/embeddings",
            "model"          : "text-embedding-3-small"
            }
```
            

Vertex AI example:
```
            {
            "provider"       : "vertexai",
            "credential_name": "VERTEXAI_CRED",
            "url"            : "https://LOCATION-aiplatform.googleapis.com/v1/projects/PROJECT/locations/LOCATION/publishers/google/models/",
            "model"          : "textembedding-gecko:predict"
            }
```
            

The generated embedding may appear as:
```
    EMBEDDING
    -------------------------------------------------------------------------------------------------------------------------------------
    [8.78423732E-003,-4.29633334E-002,-5.93001908E-003,-4.65480909E-002,2.14333013E-002,6.53376281E-002,-5.93746938E-002,2.10403297E-002,
    4.38376889E-002,5.22960871E-002,1.25104953E-002,6.49512559E-002,-9.26998071E-003,-6.97442219E-002,-3.02916039E-002,-4.74979728E-003,
    -1.08755399E-002,-4.63751052E-003,3.62781435E-002,-9.35919806E-002,-1.13934642E-002,-5.74270077E-002,-1.36667723E-002,2.42995787E-002,
    -6.96804151E-002,4.93822657E-002,1.01460628E-002,-1.56464987E-002,-2.39410568E-002,-4.27529104E-002,-5.65665103E-002,-1.74160264E-002,
    5.05326502E-002,4.31500375E-002,-2.6994409E-002,-1.72731467E-002,9.30535868E-002,6.85951149E-004,5.61876409E-003,-9.0233935E-003,
    -2.55788807E-002,-2.04174276E-002,3.74175981E-002,-1.67872179E-002,1.07479304E-001,-6.64602639E-003,-7.65537247E-002,-9.71965566E-002,
    -3.99636962E-002,-2.57076006E-002,-5.62455431E-002,-1.3583754E-001,3.45946029E-002,1.85191762E-002,3.01524661E-002,-2.62163244E-002,
    -4.05582506E-003,1.72979087E-002,-3.66434865E-002,-1.72491539E-002,3.95228416E-002,-1.05518714E-001,-1.27463877E-001,1.42578809E-002
```
    




This example uses the default settings for each provider. For detailed information on additional parameters, refer to your third-party provider's documentation.

**Related Topics**

  * [UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector.md#GUID-8E615832-F6C0-4435-8F43-3FAF80692D5B)
  * [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12)
  * [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504)



**Parent topic:** [Generate Embeddings](generate-embeddings.md)
