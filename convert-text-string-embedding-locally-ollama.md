## Convert Text String to Embedding Using the Local REST Provider Ollama {#GUID-A77CD2EC-7DF5-4E4C-8AD8-4C274E5BA8E8}

Perform a text-to-embedding transformation by accessing open embedding models, using the local host REST endpoint provider Ollama.

Ollama is a free and open-source command-line interface tool that allows you to run open embedding models and LLMs locally and privately on your Linux, Windows, or macOS systems. You can access Ollama as a service using SQL and PL/SQL commands. 

You can call embedding models, such as all-minilm, mxbai-embed-large, or nomic-embed-text to vectorize your data. Note that you must use the same embedding model on both the data to be indexed and the user's input query. In this example, you can see how to vectorize a user's input query on the fly.

Here, you can call the chainable utility function `UTL_TO_EMBEDDING` (note the singular "embedding") from either the `DBMS_VECTOR` or the `DBMS_VECTOR_CHAIN` package, depending on your use case. This example uses the `DBMS_VECTOR.UTL_TO_EMBEDDING` API. `UTL_TO_EMBEDDING` directly returns a `VECTOR` type (not an array). 

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

To convert a user's input text string "`hello`" to a query vector, by calling a local embedding model using Ollama: 

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
        

  2. Install Ollama and run an embedding model locally.
    1. Download and run the Ollama application from <https://ollama.com/download>.

You can either install Ollama as a service that runs in the background or as a standalone binary with a manual install. For detailed installation-specific steps, see Quick Start in the [Ollama Documentation](https://github.com/ollama/ollama/tree/main/docs). 

**Note the following**: 

       * The Ollama server needs to be able to connect to the internet so that it can download the models. If you require a proxy server to access the internet, remember to set the appropriate environment variables before running the Ollama server. For example, to set for Linux: 
```
            -- set a proxy if you require one
            
            export https_proxy=:
            export http_proxy=:
            export no_proxy=localhost,127.0.0.1,.example.com
            export ftp_proxy=:
```
            

       * If you are running Ollama and the database on different machines, then on the database machine, you must change the `URL` to refer to the host name or IP address that is running Ollama instead of the local host. 

       * You may need to change your firewall settings on the machine that is running Ollama to allow the port through.

    2. If running Ollama as a standalone binary from a manual install, then start the server:
```
        ollama serve
```
        

    3. Run a model using the `ollama pull *``*` command.

For example, to call the all-minilm model:
```
        ollama pull all-minilm
```
        

For detailed information on this step, see [Ollama Readme](https://github.com/ollama/ollama/blob/main/README.md#quickstart). 

    4. Verify that Ollama is running locally by using a cURL command.

For example:
```
        -- get embeddings
        
        curl http://localhost:11434/api/embeddings -d '{
        "model" : "all-minilm",
        "prompt": "What is Oracle AI Vector Search?"}'
```
        

  3. Set the HTTP proxy server, if configured.
```
    EXEC UTL_HTTP.SET_PROXY(':');
```
    

  4. Grant connect privilege to `docuser` for allowing connection to the host, using the `DBMS_NETWORK_ACL_ADMIN` procedure.

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
    

  5. Call `UTL_TO_EMBEDDING`.

The Ollama service has a REST API endpoint for generating embedding. Specify the URL and other configuration parameters in a JSON object.
```
    var embed_ollama_params clob;
    exec :embed_ollama_params := '{
    "provider": "ollama",
    "host"    : "local",
    "url"     : "http://localhost:11434/api/embeddings",
    "model"   : "all-minilm"
    }';
    
    select dbms_vector.utl_to_embedding('hello', json(:embed_ollama_params)) ollama_output from dual;
```
    

You can replace the `url` and `model` with your own values, as required. 

> **note:** For a complete list of all supported REST endpoint URLs, see [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 

An excerpt from the generated embedding output is as follows:
```
    OLLAMA_OUTPUT
    --------------------------------------------------------------------------------
    [-2.31221581E+000,-3.26045007E-001,2.48111725E-001,-1.65610778E+000,1.10871601E+
    000,-1.78519666E-001,-2.44365096E+000,-3.32534742E+000,-1.3771069E+000,1.8842382
    4E+000,1.26494539E+000,-2.05359578E+000,-1.78593469E+000,-3.16150457E-001,-5.362
    42545E-001,6.42113638E+000,-2.36518741E+000,2.21405053E+000,6.52316332E-001,-7.8
    1692028E-001,2.32031775E+000,5.31627655E-001,-5.02781868E-001,7.03743398E-001,5.
    48391223E-001,-3.16579014E-001,5.28999329E+000,1.63369191E+000,1.34206653E-001,9
    .54429448E-001,-1.94197679E+000,2.39797616E+000,3.5270077E-001,-1.6536833E+000,-
    5.74707508E-001,1.60994816E+000,3.80332375E+000,-6.30351126E-001,-1.5865227E+000
    ,-2.48650503E+000,-1.42142653E+000,-2.79453158E+000,1.76355612E+000,-2.48690337E
    -001,1.5521245E+000,-1.95240334E-001,1.42441893E+000,-3.57098508E+000,4.02083158
    E+000,-2.38530707E+000,2.34579134E+000,-2.79158998E+000,-5.92314243E-001,-9.7153
    
    OLLAMA_OUTPUT
    --------------------------------------------------------------------------------
    9557E-001,1.6514441E-002,1.03710043E+000,1.96799666E-001,-2.18394065E+000,-2.786
    71598E+000,-1.1549623E+000,1.92903787E-001,7.72498465E+000,-1.63462329E+000,3.33
    839393E+000,-7.17389703E-001,-3.99817854E-001,7.7395606E-001,6.43829286E-001,1.8
    5182285E+000,2.95272923E+000,-8.72635692E-002,1.77895337E-001,-1.19716954E+000,9
    .43063736E-001,1.51703429E+000,-7.93301344E-001,1.92319798E+000,3.33290434E+000,
    -1.29852867E+000,2.02961493E+000,-4.35824203E+000,1.30186975E+000,-1.0979414E+00
    0,-7.07521975E-001,-4.50183332E-001,3.77224755E+000,-2.49138021E+000,-1.12575901
    E+000,1.15370192E-001,1.66767395E+000,3.52229095E+000,-3.78049397E+000,-6.171851
    75E-001,1.71992481E+000,2.33226371E+000,-3.35983014E+000,-3.78417182E+000,3.8380
    127E+000,2.59293962E+000,-1.33574629E+000,-1.76218402E+000,3.53816301E-001,-1.80
    47899E+000,9.85167921E-001,1.93026745E+000,2.15959966E-001,9.94020045E-001,6.189
```
    




**Related Topics**

  * [UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector.md#GUID-8E615832-F6C0-4435-8F43-3FAF80692D5B)
  * [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12)
  * [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504)



**Parent topic:** [Generate Embeddings](generate-embeddings.md)
