## Access Third-Party Models for Vector Generation Leveraging Third-Party REST APIs {#GUID-F3C6AEAE-2995-4867-BE94-30907D6087BC}

You can access third-party vector embedding models to generate vector embeddings from your data, outside the database by calling third-party REST APIs.

The Vector Utility PL/SQL packages `DBMS_VECTOR` and `DBMS_VECTOR_CHAIN` (outlined in [Supplied Vector Utility PL/SQL Packages](supplied-vector-utility-pl-sql-packages.md#GUID-D73320C0-C2E3-4B67-A7C9-C0490DC9DE6C)) provide the third-party REST APIs that let you interact with external embedding models. 

You can remotely access third-party service providers, such as Cohere, Google AI, Hugging Face, Generative AI, OpenAI, and Vertex AI. Alternatively, you can install Ollama on your local host to access open LLMs, such as Llama 3, Phi 3, Mistral, and Gemma 2. All the supported providers and corresponding REST operations allowed for each provider are listed in [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 

Review these high-level steps involved in generating embeddings by calling third-party REST APIs:

  1. **Understand the terms of using third-party embedding models**. 

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs.

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

  2. **Configure your REST API connection**. 

By configuring your REST API connection, you enable Oracle Database to communicate with the REST endpoint URL where you want to send requests to access data from the third-party embedding service. The requested embedding service then processes the data and returns a vector representation.

This involves the following tasks:

    1. **Create a user, set up storage, and grant necessary privileges**. 

Create a tablespace and a user, and then grant the `DB_DEVELOPER_ROLE` to that user. This role assigns all basic roles and privileges that are necessary for a database developer. 

You can pass the input directly as a text string, or prepare a data dump directory to upload your data for vector generation. Ensure to grant the user access to read and write your data dump directory. 

    2. **Grant the connect privilege to allow connection to the third-party host**. 

Grant the `CONNECT` privilege to the user for connecting to your third-party host. You use the `DBMS_NETWORK_ACL_ADMIN` PL/SQL procedure (as described in [DBMS_NETWORK_ACL_ADMIN](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-254AE700-B355-4EBC-84B2-8EE32011E692)) to specify the users and their privilege assignments that can access external network services from within the database. 

This procedure appends an access control entry (ACE) to the network access control list (ACL) for the specified host. The ACE grants a privilege to a principal (user name), which enables the user to connect to the external host that has been authorized in the database's networking ACL. This increases security by controlling the users that can connect to the specified hosts, and thus prevents unauthorized connections.

    3. **Set the proxy server, if configured**. 

If you have configured a proxy server on your network, then you must specify the proxy server's host name and port number. This helps you access external web resources or embedding services by routing requests through your proxy server.

You use the `UTL_HTTP.SET_PROXY` PL/SQL procedure (as described in [UTL_HTTP.SET_PROXY](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ARPLS-GUID-C02A2B96-90DD-455B-A0A9-F68DB5CB3E82)) to set the proxy server. 

    4. **Set up credentials to enable access to the REST provider**. 

You require authentication credentials to enable access to your chosen third-party service provider. A credential name holds authentication parameters, such as user name, password, access token, private key, or fingerprint.

You use the credential helper procedures `CREATE_CREDENTIAL` and `DROP_CREDENTIAL` to securely manage credentials in the database. These procedures are available with both the `DBMS_VECTOR` and `DBMS_VECTOR_CHAIN` PL/SQL packages. 

  3. **Generate vector embeddings**. 

You use chainable utility functions `UTL_TO_EMBEDDING` and `UTL_TO_EMBEDDINGS` to convert input data to one or more vector embeddings. These APIs are available with both the `DBMS_VECTOR` and the `DBMS_VECTOR_CHAIN` PL/SQL packages. 

Determine which API to use:

     * `UTL_TO_EMBEDDING` converts plain text (`CLOB`) or image (`BLOB`) to a single embedding (`VECTOR`). 

     * `UTL_TO_EMBEDDINGS` converts an array of chunks (`VECTOR_ARRAY_T`) to an array of embeddings (`VECTOR_ARRAY_T`). 

For example, a typical `DBMS_VECTOR.UTL_TO_EMBEDDING` call specifies the REST provider, credential name, REST endpoint URL for an embedding service, and embedding model name parameters in a JSON object, as follows: 
```
    var params clob;
    exec :params := '
    {
    "provider": "cohere",
    "credential_name": "COHERE_CRED",
    "url": "https://api.cohere.example.com/embed",
    "model": "embed-model"
    }';
    
    select DBMS_VECTOR.UTL_TO_EMBEDDING('hello', json(:params)) from dual;
```
    

In this example, the input for generating embedding is `hello`. 




**Run end-to-end embedding examples**: 

To see how to apply these steps for performing various embedding use cases, you can run some end-to-end example scenarios listed in [Vector Generation Examples](vector-generation-examples.md#GUID-843E4921-A390-41F8-8ED0-91D7B67007B6). 

**Related Topics**

  * [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12)
  * [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504)



**Parent topic:** [Generate Vector Embeddings](generate-vector-embeddings-node.md)
