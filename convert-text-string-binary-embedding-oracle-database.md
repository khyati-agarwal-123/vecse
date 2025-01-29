## Convert Text String to BINARY Embedding Outside Oracle Database {#GUID-425922B4-C9DA-48B6-B98F-7846D53AE0AF}

Perform a text-to-`BINARY`-embedding transformation by accessing a third-party `BINARY` vector embedding model. 

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

To generate a vector embedding with "`hello`" as the input using Cohere ubinary embed-english-v3.0 model: 

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
    

  3. Grant connect privilege to allow connection to the host.

Grant connect privilege to `docuser` for connecting to the host, using the `DBMS_NETWORK_ACL_ADMIN` procedure. This example uses `*` to allow any host. However, you can explicitly specify the host that you want to connect to. 
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
    

  4. Set up your credentials for the REST provider (in this case, Cohere) and then call `UTL_TO_EMBEDDING`.

    1. Run `DBMS_VECTOR.CREATE_CREDENTIAL` to create and store a credential. 

Cohere requires the following authentication parameter:
```
        { "access_token": "" }
```
        

Replace `` with your own values. You will later refer to this credential name when declaring JSON parameters for the `UTL_TO_EMBEDDING` call. 
```
        EXEC DBMS_VECTOR.DROP_CREDENTIAL('COHERE_CRED');
        
        DECLARE
        jo json_object_t;
        BEGIN
        jo := json_object_t();
        jo.put('access_token', '');
        DBMS_VECTOR.CREATE_CREDENTIAL(
        credential_name   => 'COHERE_CRED',
        params            => json(jo.to_string));
        END;
        /
```
        

    2. Call `DBMS_VECTOR.UTL_TO_EMBEDDING` to generate the `BINARY` embedding. 

> **note:** For a list of all supported REST endpoints, see [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 
```
        var params clob;
        
        BEGIN
        :params := '
        {
        "provider": "cohere",
        "credential_name": "COHERE_CRED",
        "url": "https://api.cohere.ai/v1/embed",
        "model": "embed-english-v3.0",
        "input_type": "search_query",
        "embedding_types": ["ubinary"]
        }';
        END;
        /
        
        SELECT TO_VECTOR(FROM_VECTOR(DBMS_VECTOR.UTL_TO_EMBEDDING('hello', JSON(:params))),*,BINARY);
```
        

The generated `BINARY` embedding appears as follows: 
```
    TO_VECTOR(FROM_VECTOR(DBMS_VECTOR.UTL_TO_EMBEDDING('HELLO',JSON(:PARAMS))),*,BIN
    --------------------------------------------------------------------------------
    [137,218,245,195,211,132,169,63,43,22,12,93,112,93,85,208,145,27,76,245,99,222,1
    21,63,1,161,200,24,1,30,202,233,208,2,113,27,119,78,123,192,132,115,187,146,58,1
    36,40,63,221,52,68,241,53,88,20,99,85,248,114,177,100,248,100,158,94,53,57,97,18
    2,129,14,64,173,236,107,109,37,195,173,49,128,113,204,183,158,55,139,10,205,65,4
    0,53,243,247,134,63,125,133,55,230,129,64,165,103,102,46,251,164,213,139,227,225
    ,66,98,112,100,64,145,98,80,97,192,149,77,43,114,146,197]
```
    




**Related Topics**

  * [UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector.md#GUID-8E615832-F6C0-4435-8F43-3FAF80692D5B)



**Parent topic:** [Generate Embeddings](generate-embeddings.md)
