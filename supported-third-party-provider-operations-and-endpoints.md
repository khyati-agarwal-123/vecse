## Supported Third-Party Provider Operations and Endpoints {#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09}

Review a list of third-party REST providers and REST endpoints that are supported for various vector generation, summarization, text generation, and reranking operations.

  * [Public or Remote REST Endpoint Providers](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09__SECTION_CTG_SMC_YCC)

  * [Local REST Endpoint Provider](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09__SECTION_M1S_SMC_YCC)

  * [REST Operations](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09__SECTION_IW1_TMC_YCC)

  * [REST Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09__SECTION_T2M_TMC_YCC)

  * [Models Supported for Generative AI](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09__TITLE_ISR_31X_2DC)




Public or Remote REST Endpoint Providers

The supported publicly-hosted, third-party REST endpoint providers are:

  * Cohere

  * Google AI

  * Hugging Face

  * Oracle Cloud Infrastructure (OCI) Generative AI

  * OpenAI

  * Vertex AI




Local REST Endpoint Provider

You can use Ollama as a third-party REST endpoint provider, locally and privately on your Linux, Windows, and macOS systems. 

Ollama is a free and open-source command-line interface tool that allows you to run open LLMs (such as Llama 3, Phi 3, Mistral, and Gemma 2) and embedding models (such as mxbai-embed-large, nomic-embed-text, or all-minilm). You can access Ollama using SQL and PL/SQL commands.

You can download and run the Ollama application from <https://ollama.com/download>. You can either install Ollama as a service that runs in the background or as a standalone binary with a manual install. For detailed installation-specific steps, see Quick Start in the [Ollama Documentation](https://github.com/ollama/ollama/tree/main/docs). 

REST Operations

These are the supported third-party REST operations and APIs along with their corresponding REST providers:

Operation | Provider | API  
---|---|---  
Generate embedding: <br>Convert textual documents and images to one or more vector embeddings | 

  * For text input: <br>All supported public and local providers <br>For image input: <br>Vertex AI

| 

  * [DBMS_VECTOR.UTL_TO_EMBEDDING and DBMS_VECTOR.UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector.md#GUID-8E615832-F6C0-4435-8F43-3FAF80692D5B) <br>[DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDING and DBMS_VECTOR_CHAIN.UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector_chain.md#GUID-C6439E94-4E86-4ECD-954E-4B73D53579DE)

  
Generate summary: <br>Extract a brief and comprehensive summary from textual documents |  All supported public and local providers | 

  * [DBMS_VECTOR_CHAIN.UTL_TO_SUMMARY](utl_to_summary.md#GUID-EC9DDB58-6A15-4B36-BA66-ECBA20D2CE57)

  
Generate text: <br>Retrieve a descriptive response for textual prompts and images through conversations with LLMs | 

  * For text input: <br>All supported public and local providers <br>For image input: <br>Google AI, Hugging Face, OpenAI, Ollama, and Vertex AI

| 

  * [DBMS_VECTOR.UTL_TO_GENERATE_TEXT](utl_to_generate_text-dbms_vector.md#GUID-EA78DFB6-D951-43D1-8ECB-DD6D21C6F6A6) <br>[DBMS_VECTOR_CHAIN.UTL_TO_GENERATE_TEXT](utl_to_generate_text-dbms_vector_chain.md#GUID-017C9002-194C-48E5-B59B-EF5C60BC8405)

  
Rerank results: <br>Reassess and reorder search results to retrieve a more relevant output |  Cohere and Vertex AI | 

  * [DBMS_VECTOR.RERANK](rerank-dbms_vector.md#GUID-E821AF93-C180-4CEB-BE4F-2339B07ED2FE) <br>[DBMS_VECTOR_CHAIN.RERANK](rerank-dbms_vector_chain.md#GUID-08B0E5EE-B097-43CD-828C-05D45B70157D)

  
  
REST Endpoints

These are the supported REST endpoints for all third-party REST providers:

API | Provider | Endpoint  
---|---|---  
`UTL_TO_EMBEDDING` <br>and<br>`UTL_TO_EMBEDDINGS` |  Cohere |  `https: <br>//api.cohere.ai/v1/embed`  
Generative AI |  `https: <br>//inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/embedText`  
Google AI |  `https: <br>//generativelanguage.googleapis.com/v1beta/models/`  
Hugging Face |  `https: <br>//api-inference.huggingface.co/pipeline/feature-extraction/`  
Ollama |  `http: <br>//localhost: <br>11434/api/embeddings`  
OpenAI |  `https: <br>//api.openai.com/v1/embeddings`  
Vertex AI |  `https: <br>//LOCATION-aiplatform.googleapis.com/v1/projects/PROJECT/locations/LOCATION/publishers/google/models/`  
`UTL_TO_SUMMARY` |  Cohere |  `https: <br>//api.cohere.ai/v1/chat `<br>`https: <br>//api.cohere.ai/v1/summarize `  
Generative AI |  `https: <br>//inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat`  
Google AI  |  `https: <br>//generativelanguage.googleapis.com/v1beta/models/`  
Hugging Face |  `https: <br>//api-inference.huggingface.co/models/`  
Ollama |  `http: <br>//localhost: <br>11434/api/generate`  
OpenAI |  `https: <br>//api.openai.com/v1/chat/completions`<br>`https: <br>//api.openai.com/v1/completions`  
Vertex AI |  `https: <br>//LOCATION-aiplatform.googleapis.com/v1/projects/PROJECT/locations/LOCATION/publishers/google/models/`  
`UTL_TO_GENERATE_TEXT` |  Cohere |  `https: <br>//api.cohere.ai/v1/chat`<br>`https: <br>//api.cohere.ai/v1/generate`  
Generative AI |  `https: <br>//inference.generativeai.us-chicago-1.oci.oraclecloud.com/20231130/actions/chat`  
Google AI |  `https: <br>//generativelanguage.googleapis.com/v1beta/models/`  
Hugging Face |  `https: <br>//api-inference.huggingface.co/models/`  
Ollama |  `http: <br>//localhost: <br>11434/api/generate`  
OpenAI  |  `https: <br>//api.openai.com/v1/chat/completions`<br>`https: <br>//api.openai.com/v1/completions`  
Vertex AI |  `https: <br>//LOCATION-aiplatform.googleapis.com/v1/projects/PROJECT/locations/LOCATION/publishers/google/models/`  
`RERANK` |  Cohere |  `https: <br>//api.cohere.com/v1/rerank`  
Vertex AI |  `https: <br>//discoveryengine.googleapis.com/v1/projects/PROJECT/locations/global/rankingConfigs/default_ranking_config: <br>rank`  
  
Models Supported for Generative AI

These are the third-party models that are supported to use with Generative AI corresponding to each REST API:

API | Model  
---|---  
`UTL_TO_EMBEDDING` <br>and<br>`UTL_TO_EMBEDDINGS` |  cohere.embed-english-v3.0<br>cohere.embed-multilingual-v3.0<br>cohere.embed-english-light-v3.0<br>cohere.embed-multilingual-light-v3.0  
`UTL_TO_SUMMARY` |  cohere.command-r-16k <br>cohere.command-r-plus<br>meta.llama-3.1-70b-instruct <br>meta.llama-3.1-405b-instruct  
`UTL_TO_GENERATE_TEXT` |  cohere.command-r-16k <br>cohere.command-r-plus<br>meta.llama-3.1-70b-instruct <br>meta.llama-3.1-405b-instruct  
  
**Parent topic:** [Work with LLM-Powered APIs and Retrieval Augmented Generation](work-llm-powered-apis-and-retrieval-augmented-generation-node.md)
