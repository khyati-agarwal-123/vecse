## Generate Summary Using Public REST Providers {#GUID-915112EE-3681-4765-8B3F-F21313EE4878}

Perform a text-to-summary transformation, using publicly hosted third-party text summarization models by Cohere, Generative AI, Google AI, Hugging Face, OpenAI, or Vertex AI.

Here, you call the chainable utility function `UTL_TO_SUMMARY`. 

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

To summarize a textual extract, using an external LLM:

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
        

  2. Set proxy if one exists.
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
    

  4. Set up your credentials for the REST provider that you want to access and then call `UTL_TO_SUMMARY`.

     * **Using Generative AI**: 

> **note:** Currently, `UTL_TO_SUMMARY` does not work for Generative AI because the model and summary endpoint supported for Generative AI have been retired. It will be available in a subsequent release. 

       1. Run `DBMS_VECTOR_CHAIN.CREATE_CREDENTIAL` to create and store an OCI credential (`OCI_CRED`). 

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
            

You will later refer to this credential name when declaring JSON parameters for the `UTL_TO_SUMMARY` call. 

> **note:** 

The generated private key may appear as:
```
            -----BEGIN RSA PRIVATE KEY-----
            
            -----END RSA PRIVATE KEY-----
```
            

You pass the *``*  value (excluding the `BEGIN` and `END` lines), either as a single line or as multiple lines. 
```
            exec dbms_vector_chain.drop_credential('OCI_CRED');
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
            dbms_vector_chain.create_credential(
            credential_name   => 'OCI_CRED',
            params            => json(jo.to_string));
            end;
            /
```
            

Replace all the authentication parameter values. 

For example:
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
            dbms_vector_chain.create_credential(
            credential_name   => 'OCI_CRED',
            params            => json(jo.to_string));
            end;
            /
```
            

       2. Run `DBMS_VECTOR_CHAIN.UTL_TO_SUMMARY`: 

Here, the cohere.command-r-16k model is used. You can replace the `model` value, as required. 

> **note:** For a list of all REST endpoint URLs and models that are supported to use with Generative AI, see [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 
```
            -- select example
            
            var params clob;
            exec :params := '
            {
            "provider"       : "ocigenai",
            "credential_name": "OCI_CRED",
            "url"            : "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat",
            "model"          : "cohere.command-r-16k"
            }';
            
            select dbms_vector_chain.utl_to_summary(
            'A transaction is a logical, atomic unit of work that contains one or more SQL
            statements.
            An RDBMS must be able to group SQL statements so that they are either all
            committed, which means they are applied to the database, or all rolled back, which
            means they are undone.
            An illustration of the need for transactions is a funds transfer from a savings account to
            a checking account. The transfer consists of the following separate operations:
            1. Decrease the savings account.
            2. Increase the checking account.
            3. Record the transaction in the transaction journal.
            Oracle Database guarantees that all three operations succeed or fail as a unit. For
            example, if a hardware failure prevents a statement in the transaction from executing,
            then the other statements must be rolled back.
            Transactions set Oracle Database apart from a file system. If you
            perform an atomic operation that updates several files, and if the system fails halfway
            through, then the files will not be consistent. In contrast, a transaction moves an
            Oracle database from one consistent state to another. The basic principle of a
            transaction is all or nothing: an atomic operation succeeds or fails as a whole.',
            json(:params)) from dual;
            
            -- PL/SQL example
            
            declare
            input clob;
            params clob;
            output clob;
            begin
            input := 'A transaction is a logical, atomic unit of work that contains one or more SQL
            statements.
            An RDBMS must be able to group SQL statements so that they are either all
            committed, which means they are applied to the database, or all rolled back, which
            means they are undone.
            An illustration of the need for transactions is a funds transfer from a savings account to
            a checking account. The transfer consists of the following separate operations:
            1. Decrease the savings account.
            2. Increase the checking account.
            3. Record the transaction in the transaction journal.
            Oracle Database guarantees that all three operations succeed or fail as a unit. For
            example, if a hardware failure prevents a statement in the transaction from executing,
            then the other statements must be rolled back.
            Transactions set Oracle Database apart from a file system. If you
            perform an atomic operation that updates several files, and if the system fails halfway
            through, then the files will not be consistent. In contrast, a transaction moves an
            Oracle database from one consistent state to another. The basic principle of a
            transaction is all or nothing: an atomic operation succeeds or fails as a whole.';
            
            params := '
            {
            "provider"       : "ocigenai",
            "credential_name": "OCI_CRED",
            "url"            : "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat",
            "model"          : "cohere.command-r-16k"
            }';
            
            output := dbms_vector_chain.utl_to_summary(input, json(params));
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
            

Optionally, you can specify additional REST provider-specific parameters. For example:
```
            {
            "provider"       : "ocigenai",
            "credential_name": "OCI_CRED",
            "url"            : "https://inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat",
            "model"          : "cohere.command-r-16k",
            "length"         : "MEDIUM",
            "format"         : "PARAGRAPH",
            "temperature"    : 1.0
            }
```
            

     * **Using Cohere, Google AI, Hugging Face, OpenAI, and Vertex AI**: 

       1. Run `DBMS_VECTOR_CHAIN.CREATE_CREDENTIAL` to create and store a credential. 

Cohere, Google AI, Hugging Face, OpenAI, and Vertex AI require the following authentication parameter: 

`{ "access_token": "*``*" }`

You will later refer to this credential name when declaring JSON parameters for the `UTL_TO_SUMMARY` call. 
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
            

Replace the `access_token` and `credential_name` values. For example: 
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
            

       2. Run `DBMS_VECTOR_CHAIN.UTL_TO_SUMMARY`: 
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
            
            select dbms_vector_chain.utl_to_summary(
            'A transaction is a logical, atomic unit of work that contains one or more SQL
            statements.
            An RDBMS must be able to group SQL statements so that they are either all
            committed, which means they are applied to the database, or all rolled back, which
            means they are undone.
            An illustration of the need for transactions is a funds transfer from a savings account to
            a checking account. The transfer consists of the following separate operations:
            1. Decrease the savings account.
            2. Increase the checking account.
            3. Record the transaction in the transaction journal.
            Oracle Database guarantees that all three operations succeed or fail as a unit. For
            example, if a hardware failure prevents a statement in the transaction from executing,
            then the other statements must be rolled back.
            Transactions set Oracle Database apart from a file system. If you
            perform an atomic operation that updates several files, and if the system fails halfway
            through, then the files will not be consistent. In contrast, a transaction moves an
            Oracle database from one consistent state to another. The basic principle of a
            transaction is "all or nothing": an atomic operation succeeds or fails as a whole.',
            json(:params)) from dual;
            
            -- PL/SQL example
            
            declare
            input clob;
            params clob;
            output clob;
            begin
            input := 'A transaction is a logical, atomic unit of work that contains one or more SQL
            statements.
            An RDBMS must be able to group SQL statements so that they are either all
            committed, which means they are applied to the database, or all rolled back, which
            means they are undone.
            An illustration of the need for transactions is a funds transfer from a savings account to
            a checking account. The transfer consists of the following separate operations:
            1. Decrease the savings account.
            2. Increase the checking account.
            3. Record the transaction in the transaction journal.
            Oracle Database guarantees that all three operations succeed or fail as a unit. For
            example, if a hardware failure prevents a statement in the transaction from executing,
            then the other statements must be rolled back.
            Transactions set Oracle Database apart from a file system. If you
            perform an atomic operation that updates several files, and if the system fails halfway
            through, then the files will not be consistent. In contrast, a transaction moves an
            Oracle database from one consistent state to another. The basic principle of a
            transaction is "all or nothing": an atomic operation succeeds or fails as a whole.';
            
            params := '
            {
            "provider": "",
            "credential_name": "",
            "url": "",
            "model": ""
            }';
            
            output := dbms_vector_chain.utl_to_summary(input, json(params));
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
            

> **note:** For a list of all supported REST endpoint URLs, see [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 

Replace `provider`, `credential_name`, `url`, and `model` with your own values. Optionally, you can specify additional REST provider parameters. This is shown in the following examples: 

Cohere example: 
```
            {
            "provider"        : "cohere",
            "credential_name" : "COHERE_CRED",
            "url"             : "https://api.cohere.ai/v1/chat",
            "model"           : "command",
            "length"          : "medium",
            "format"          : "paragraph",
            "temperature"     : 1.0
            }
```
            

Google AI example:
```
            {
            "provider"         : "googleai",
            "credential_name"  : "GOOGLEAI_CRED",
            "url"              : "https://generativelanguage.googleapis.com/v1beta/models/",
            "model"            : "gemini-pro:generateContent",
            "generation_config": {
            "temperature"    : 0.9,
            "topP"           : 1,
            "candidateCount" : 1,
            "maxOutputTokens": 256
            }
            }
```
            

Hugging Face example:
```
            {
            "provider"         : "huggingface",
            "credential_name"  : "HF_CRED",
            "url"              : "https://api-inference.huggingface.co/models/",
            "model"            : "facebook/bart-large-cnn"
            }
```
            

OpenAI example:
```
            {
            "provider"        : "openai",
            "credential_name" : "OPENAI_CRED",
            "url"             : "https://api.openai.com/v1/chat/completions",
            "model"           : "gpt-4o-mini",
            "max_tokens"      : 256,
            "temperature"     : 1.0
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
            

A generated summary may appear as:
```
    A transaction is a logical unit of work that groups one or more SQL statements
    that must be executed as a unit, with all statements succeeding, or all
    statements being rolled back. Transactions are a fundamental concept in
    relational database management systems (RDBMS), and Oracle Database is
    specifically designed to manage transactions, ensuring database consistency and
    integrity. Transactions differ from file systems in that they maintain
    atomicity, ensuring that all related operations succeed or fail as a whole,
    maintaining database consistency regardless of intermittent failures.
    Transactions move a database from one consistent state to another, and the
    fundamental principle is that a transaction is committed or rolled back as a
    whole, upholding the "all or nothing" principle.
    
    PL/SQL procedure successfully completed.
```
    




This example uses the default settings for each provider. For detailed information on additional parameters, refer to your third-party provider's documentation.

**Related Topics**

  * [About Chainable Utility Functions and Common Use Cases](chainable-utility-functions-and-common-use-cases.md#GUID-63773EF5-071E-4C02-9EC9-D4E0BA2CE2A2 "These are intended to be a set of chainable and flexible "stages" through which you pass your input data to transform into a different representation, including vectors.")
  * [UTL_TO_SUMMARY](utl_to_summary.md#GUID-EC9DDB58-6A15-4B36-BA66-ECBA20D2CE57)



**Parent topic:** [Generate Summary](generate-summary.md)
