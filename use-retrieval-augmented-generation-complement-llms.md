## Use Retrieval Augmented Generation to Complement LLMs {#GUID-469E664A-3DCF-43EB-9193-C72912F61ABA}

RAG lets you mitigate the inaccuracies and hallucinations faced when using LLMs. Oracle AI Vector Search enables RAG within Oracle Database using the `DBMS_VECTOR_CHAIN` PL/SQL package or through popular frameworks (such as LangChain). 

  * [About Retrieval Augmented Generation](retrieval-augmented-generation1.md)  
Oracle AI Vector Search supports Enterprise Retrieval Augmented Generation (RAG) to enable sophisticated queries that can combine vectors with relational data, graph data, spatial data, and JSON collections. 
  * [SQL RAG Example](sql-rag-example.md)  
This scenario allows you to run a similarity search for specific documentation content based on a user query. Once documentation chunks are retrieved, they are concatenated and a prompt is generated to ask an LLM to answer the user question using retrieved chunks. 
  * [Oracle AI Vector Search Integration with LangChain](oracle-ai-vector-search-integration-langchain.md)  
LangChain is a powerful and flexible open source orchestration framework that helps developers build applications that leverage the advanced capabilities of large language models (LLMs). 
  * [Oracle AI Vector Search Integration with LlamaIndex](oracle-ai-vector-search-integration-llamaindex.md)  
LlamaIndex is an open-source data framework designed to simplify the process of building applications that leverage large language models (LLMs) with custom data. Basically, LlamaIndex acts as a bridge between custom data sources and LLMs such as Cohere Command models or OpenAI GPTs models. 
  * [Use Reranking for Better RAG Results](use-reranking-better-rag-results.md)  
Reranking models are primarily used to reassess and reorder an initial set of search results. This helps to improve the relevance and quality of search results in both similarity search and Retrieval Augmented Generation (RAG) scenarios. 



**Parent topic:** [Work with LLM-Powered APIs and Retrieval Augmented Generation](work-llm-powered-apis-and-retrieval-augmented-generation-node.md)
