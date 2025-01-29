## Use Reranking for Better RAG Results {#GUID-969CE0B6-FF30-46F0-AADC-90B406F4F645}

Reranking models are primarily used to reassess and reorder an initial set of search results. This helps to improve the relevance and quality of search results in both similarity search and Retrieval Augmented Generation (RAG) scenarios.

In a RAG scenario, reranking plays a crucial role in improving the quality of information ingested into an LLM by ensuring that the most relevant documents or chunks are prioritized. This can reduce hallucinations and improve the accuracy of generated outputs. The reranking step is typically performed after an initial search that uses a faster but less precise embedding model. The reranker helps to identify the most pertinent information for a given query, but is more expensive in terms of resources because it often employs sophisticated matching methods that go beyond simple vector comparisons.

Here, you can use the `RERANK` function from either the `DBMS_VECTOR` or the `DBMS_VECTOR_CHAIN` package, depending on your use case. 

Oracle AI Vector Search supports reranking models provided by Cohere and Vertex AI.

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

To rerank results for the query "`What are some interesting characteristics of the Jovian satellites?`", using a third-party reranking model: 

  1. Log in to SQL*Plus as the `SYS` user, connecting as `SYSDBA`:
```
    conn sys/password as sysdba
```
    

  2. Create a local user (`vector`) and grant necessary privileges:
```
    DROP USER vector cascade;
```
```
    CREATE USER vector identified by
```
```
    GRANT DB_DEVELOPER_ROLE, CREATE CREDENTIAL TO vector;
```
    

  3. Set proxy if one exists.
```
    EXEC UTL_HTTP.SET_PROXY(':');
```
    

  4. Grant connect privilege for a host using the `DBMS_NETWORK_ACL_ADMIN` procedure. This example uses `*` to allow any host. However, you can explicitly specify each host that you want to connect to:
```
    BEGIN
    DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
    host => '*',
    ace => xs$ace_type(privilege_list => xs$name_list('connect'),
    principal_name => 'VECTOR',
    principal_type => xs_acl.ptype_db));
    END;
    /
```
    

  5. Connect to Oracle Database as the test user:
```
    conn vector/password;
```
    

  6. Simulate the initial retrieval step that returns, in the specified order, potentially relevant documents for the following query: "`What are some interesting characteristics of the Jovian satellites?`":
```
    set echo on
    
    set serveroutput on
    
    var query clob;
    
    var initial_retrieval_docs clob;
    
    exec :query := 'What are some interesting characteristics of the Jovian satellites?';
    
    begin
    :initial_retrieval_docs := '
    {
    "documents": [
    "Jupiter boasts an impressive system of 95 known moons, including the four largest Galilean satellites.",
    "Jupiter's immense mass, 318 times that of Earth, significantly influences the orbits of other bodies in the Solar System.",
    "Io, one of Jupiter's Galilean moons, is the most volcanically active body in our solar system.",
    "The gas giant completes one orbit around the Sun in just under 12 years, traveling at an average speed of 13 kilometers per second.",
    "Jupiter's composition is similar to that of the Sun, and it could have become a brown dwarf if its mass had been 80 times greater."
    ]
    }';
    end;
    /
```
    

  7. Set up your credentials for the REST provider that you want to access, that is, Cohere or Vertex AI. Here, replace *``*  with your own values:

     * Cohere example:
```
        EXEC DBMS_VECTOR.DROP_CREDENTIAL('COHERE_CRED');
        
        DECLARE
        jo json_object_t;
        BEGIN
        jo := json_object_t();
        jo.put('access_token', '');
        DBMS_VECTOR.CREATE_CREDENTIAL(
        credential_name => 'COHERE_CRED',
        params => json(jo.to_string));
        END;
        /
```
        

     * Vertex AI example:
```
        begin
        dbms_vector_chain.drop_credential(credential_name  => 'VERTEXAI_CRED');
        exception
        when others then null;
        end;
        /
        
        declare
        jo json_object_t;
        begin
        jo := json_object_t();
        jo.put('access_token', '');
        dbms_vector_chain.create_credential(
        credential_name   =>  'VERTEXAI_CRED',
        params            => json(jo.to_string));
        end;
        /
```
        

  8. Rerank the initial retrieval:

> **note:** For a list of all supported REST endpoints, see [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 

     * Using the Cohere `rerank-english-v3.0` model: 
```
        declare
        params clob;
        reranked_output json;
        begin
        params := '
        {
        "provider": "cohere",
        "credential_name": "COHERE_CRED",
        "url": "https://api.cohere.com/v1/rerank",
        "model": "rerank-english-v3.0",
        "return_documents": true,
        "top_n": 3
        }';
        
        reranked_output := dbms_vector_chain.rerank(:query, json(:initial_retrieval_docs), json(params));
        dbms_output.put_line(json_serialize(reranked_output));
        end;
        /
```
        

     * Using the Vertex AI `semantic-ranker-512` model: 
```
        declare
        params clob;
        reranked_output json;
        begin
        params := '
        {
        "provider": "vertexai",
        "credential_name": "VERTEXAI_CRED",
        "url": "https://discoveryengine.googleapis.com/v1/projects/1085581009881/locations/global/rankingConfigs/default_ranking_config:rank",
        "model": "semantic-ranker-512@latest",
        "ignoreRecordDetailsInResponse": false,
        "topN": 3
        }';
        
        reranked_output := dbms_vector_chain.rerank(:query, json(:initial_retrieval_docs), json(params));
        dbms_output.put_line(json_serialize(reranked_output));
        end;
        /
```
        




Using the above Cohere model, the reranked results appear as follows:
```
    [
    {
    "index" : "0",
    "score" : "0.059319142",
    "content" : "Jupiter boasts an impressive system of 95 known moons, including the four largest Galilean satellites."
    },
    {
    "index" : "2",
    "score" : "0.04352814",
    "content" : "Io, one of Jupiter's Galilean moons, is the most volcanically active body in our solar system."
    },
    {
    "index" : "4",
    "score" : "0.04138472",
    "content" : "Jupiter's composition is similar to that of the Sun, and it could have become a brown dwarf if its mass had been 80 times greater."
    }
    ]
```
    

**Related Topics**

  * [RERANK](rerank-dbms_vector_chain.md#GUID-08B0E5EE-B097-43CD-828C-05D45B70157D)
  * [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12)
  * [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504)



**Parent topic:** [Use Retrieval Augmented Generation to Complement LLMs](use-retrieval-augmented-generation-complement-llms.md)
