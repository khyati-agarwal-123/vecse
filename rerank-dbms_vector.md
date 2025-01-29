## RERANK {#GUID-E821AF93-C180-4CEB-BE4F-2339B07ED2FE}

Use the `DBMS_VECTOR.RERANK` function to reassess and reorder an initial set of results to retrieve more relevant search output. 

Purpose

To improve the relevance and quality of search results in both similarity search and Retrieval Augmented Generation (RAG) scenarios. 

Reranking improves the quality of information ingested into an LLM by ensuring that the most relevant documents or chunks are prioritized. This helps to reduce hallucinations and improves the accuracy of generated outputs.

For this operation, Oracle AI Vector Search supports reranking models provided by Cohere and Vertex AI.

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

Syntax
```
    DBMS_VECTOR.RERANK(
    QUERY      IN CLOB,
    DOCUMENTS  IN JSON,
    PARAMS     IN JSON default NULL
    ) return JSON;
```
    

This function accepts the input containing a query as `CLOB` and a list of documents in `JSON` format. It then processes this information to generate a `JSON` object containing a reranked list of documents, sorted by score. 

For example, a reranked output includes:
```
    {
    "index"   : "1",
    "score"   : "0.99",
    "content" : "Jupiter boasts an impressive system of 95 known moons."
    }
```
    

Where, 

  * `index` specifies the position of the document in the list of input text. 

  * `score` specifies the relevance score. 

  * `content` specifies the input text corresponding to the index. 




QUERY

Specify the search query (typically from an initial search) as `CLOB`. 

DOCUMENTS

Specify a JSON array of strings (list of potentially relevant documents to rerank) in the following format:
```
    {
    "documents": [
    "string1",
    "string2",
    ...
    ]
    }
```
    

PARAMS

Specify the following list of parameters in JSON format. All these parameters are mandatory.
```
    {
    "provider"         : "",
    "credential_name"  : "",
    "url"              : "",
    "model"            : "",
    ...
    }
```
    

**Table: RERANK Parameter Details**

Parameter | Description  
---|---  
`provider` |  Supported REST provider to access for reranking: <br>* `cohere` <br>`vertexai`  
  
`credential_name` |  Name of the credential in the form: <br> *`schema`*.*`credential_name`* <br>A credential name holds authentication credentials to enable access to your provider for making REST API calls. <br>You need to first set up your credential by calling the `DBMS_VECTOR.CREATE_CREDENTIAL` helper function to create and store a credential, and then refer to the credential name here. <br>See [CREATE_CREDENTIAL](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-4BBCBF46-3903-4EBB-8BE8-A7690151CF25).   
`url` |  URL of the third-party provider endpoint for each REST call, as listed in [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09).   
`model` |  Name of the reranking model in the form: <br> *`schema`*.*`model_name`* <br>If the model name is not schema-qualified, then the schema of the procedure invoker is used.  
  
**Additional REST provider parameters**: 

Optionally, specify additional provider-specific parameters for reranking.

> **note:** Important: 

  * The following examples are for illustration purposes. For accurate and up-to-date information on additional parameters to use, refer to your third-party provider's documentation.

  * For a list of all supported REST endpoints, see [Supported Third-Party Provider Operations and Endpoints](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 




Cohere example:
```
    {
    "provider"        : "cohere",
    "credential_name" : "COHERE_CRED",
    "url"             : "https://api.cohere.example.com/rerank",
    "model"           : "rerank-english-v3.0",
    "return_documents": false,
    "top_n"           : 3
    }
```
    

Vertex AI example:
```
    {
    "provider"         : "vertexai",
    "credential_name"  : "VERTEXAI_CRED",
    "url"              : "https://googleapis.example.com/default_ranking_config:rank",
    "model"            : "semantic-ranker-512@latest",
    "ignoreRecordDetailsInResponse" : true,
    "topN"             : 3
    }
```
    

**Table: Additional REST Provider Parameter Details**

Parameter | Description  
---|---  
`return_documents` |  Whether to return search results with original documents or input text (`content`): <br>* `false` (default, also recommended) to not return any input text (return only index and score)  <br>`true` to return input text along with index and score 
<br>**Note**: <br>With Cohere as the provider, Oracle recommends that you keep this option disabled for better performance. You may choose to enable it for debugging purposes when you need to view the original text.   
`ignoreRecordDetailsInResponse` |  Whether to return search results with original record details or input text (`content`): <br>* `false` (default) to return input text along with index and score  <br>`true` (recommended) to not return any input text (return only index and score) 
<br>**Note**: <br>With Vertex AI as the provider, Oracle recommends that you keep this option enabled for better performance. You may choose to disable it for debugging purposes when you need to view the original text.   
`top_n` or `topN` |  The number of most relevant documents to return.  
  
Examples

  * Using Cohere:
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
    
    reranked_output := dbms_vector.rerank(:query, json(:initial_retrieval_docs), json(params));
    dbms_output.put_line(json_serialize(reranked_output));
    end;
    /
```
    

  * Using Vertex AI:
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
    
    reranked_output := dbms_vector.rerank(:query, json(:initial_retrieval_docs), json(params));
    dbms_output.put_line(json_serialize(reranked_output));
    end;
    /
```
    




**End-to-end example**: 

To run an end-to-end example scenario using this function, see [Use Reranking for Better RAG Results](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-969CE0B6-FF30-46F0-AADC-90B406F4F645). 

**Parent topic:** [DBMS_VECTOR](dbms_vector-vecse.md)
