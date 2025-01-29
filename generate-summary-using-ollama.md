## Generate Summary Using the Local REST Provider Ollama {#GUID-26A150D2-8B3A-4785-B8AF-88C9BA6B51EC}

Perform a text-to-summary transformation by accessing open LLMs, using the local host REST endpoint provider Ollama.

Ollama is a free and open-source command-line interface tool that allows you to run open LLMs (such as Llama 3, Phi 3, Mistral, or Gemma 2) locally and privately on your Linux, Windows, or macOS systems. You can access Ollama as a service using SQL and PL/SQL commands.

Here, you call the chainable utility function `UTL_TO_SUMMARY`. 

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

To generate a concise and informative summary of a textual extract, by calling a local LLM using Ollama:

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
        

  2. Install Ollama and run a model locally.
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
        

    3. Run a model using the `ollama run *``*` command.

For example, to call the Llama 3 model:
```
        ollama run llama3
```
        

For detailed information on this step, see [Ollama Readme](https://github.com/ollama/ollama/blob/main/README.md#quickstart). 

    4. Verify that Ollama is running locally by using a cURL command.

For example:
```
        -- get summary
        
        curl http://localhost:11434/api/generate -d '{
        "model" : "llama3",
        "prompt": "What is Oracle AI Vector Search?"
        "stream": false}'
```
        

  3. Set proxy if one exists.
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
    

  5. Call `UTL_TO_SUMMARY`.

The Ollama service has a REST API endpoint for summarizing text. Specify the URL and other configuration parameters in a JSON object.
```
    var gent_ollama_params clob;
    exec :gent_ollama_params := '{
    "provider": "ollama",
    "host"    : "local",
    "url"     : "http://localhost:11434/api/generate",
    "model"   : "llama3"
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
    json(:gent_ollama_params)) from dual;
```
    

You can replace the `url` and `model` with your own values, as required. 

> **note:** For a complete list of all supported REST endpoint URLs, see [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 

A generated summary may appear as:
```
    A transaction in an RDBMS (Relational Database Management System) is a self-contained unit of work that
    consists of one or more SQL statements. This ensures that all changes made by the transaction are either
    committed (applied to the database) or rolled back (undone). Oracle Database is specifically designed to
    manage transactions, ensuring database consistency and integrity.
    Transactions differ from file systems in that they maintain atomicity, ensuring that all related operations
    succeed or fail as a whole, maintaining database consistency regardless of intermittent failures.
    Transactions move a database from one consistent state to another, and the fundamental principle is that a
    transaction is committed or rolled back as a whole, upholding the "all or nothing" principle.
```
    




**Related Topics**

  * [About Chainable Utility Functions and Common Use Cases](chainable-utility-functions-and-common-use-cases.md#GUID-63773EF5-071E-4C02-9EC9-D4E0BA2CE2A2 "These are intended to be a set of chainable and flexible "stages" through which you pass your input data to transform into a different representation, including vectors.")
  * [UTL_TO_SUMMARY](utl_to_summary.md#GUID-EC9DDB58-6A15-4B36-BA66-ECBA20D2CE57)



**Parent topic:** [Generate Summary](generate-summary.md)
