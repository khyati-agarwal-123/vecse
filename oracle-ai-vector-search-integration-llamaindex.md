## Oracle AI Vector Search Integration with LlamaIndex {#GUID-C28D7056-E84E-4669-8155-092EBB4F9AB3}

LlamaIndex is an open-source data framework designed to simplify the process of building applications that leverage large language models (LLMs) with custom data. Basically, LlamaIndex acts as a bridge between custom data sources and LLMs such as Cohere Command models or OpenAI GPTs models. 

Oracle AI Vector Search is integrated with LlamaIndex in several ways to enable powerful semantic search and retrieval capabilities. 

Here are the key aspects of this integration:

  * **Embedding Generation**

Oracle AI Vector Search provides embedding capabilities that can be used with LlamaIndex: 

    * The `OracleEmbeddings` class from LlamaIndex can be used to generate embeddings using Oracle's embedding models. 

    * Multiple embedding methods are supported, including locally-hosted ONNX models and third-party APIs such as Generative AI and Hugging Face.

    * Embeddings can be generated for documents and queries to enable semantic similarity search.

For more information on how to use this integration for generating embeddings, see [Oracle AI Vector Search: Use Embedding Generation Capabilities](https://github.com/run-llama/llama_index/blob/main/docs/docs/examples/embeddings/oracleai.ipynb). 

  * **Vector Storage**

LlamaIndex can leverage Oracle Database as a vector store: 

    * Vector embeddings can be stored alongside business data in Oracle Database tables using the `VECTOR` data type. 

    * This allows combining semantic search on unstructured data with relational queries on structured data in a single system.

For more information on how to use this integration for vector storage, see [Oracle AI Vector Search: Use Vector Storage Capabilities](https://github.com/run-llama/llama_index/blob/main/docs/docs/examples/vector_stores/orallamavs.ipynb). 

  * **Indexing and Search**

You can utilize Oracle's vector indexing and search capabilities:

    * Vector indexes can be created on the embeddings to enable fast similarity search.

    * LlamaIndex can use Oracle's native SQL operations for similarity search to retrieve relevant data.

    * Various distance metrics, such as dot product, cosine similarity, Euclidean distance, and more are supported.

For more information about how to use this integration for indexing and search, see [Oracle AI Vector Search: Use Document Processing Capabilities](https://github.com/run-llama/llama_index/blob/main/docs/docs/examples/data_connectors/oracleai.ipynb) and [Oracle AI Vector Search: End-to-End Pipeline with Document Processing](https://github.com/run-llama/llama_index/blob/main/docs/docs/examples/cookbooks/oracleai_demo.ipynb). 

  * **RAG Pipeline Integration**

The integration enables building end-to-end Retrieval Augmented Generation (RAG) pipelines. You can embed and store unstructured data in Oracle Database. LlamaIndex can query the vector store to retrieve relevant context. The retrieved information can be used to generate prompts for LLMs.

LlamaIndex provides several libraries and classes to integrate Oracle AI Vector Search capabilities. Here are the key components available: 

    * `OracleEmbeddings`: Supports multiple embedding methods, including locally hosted ONNX models and third-party APIs such as Generative AI and Hugging Face. 

    * `OracleReader`: Used for loading documents from various sources, including Oracle Database. 

    * `OracleSummary`: Provides functionality for summarizing documents within or outside the database. 

    * `OracleTextSplitter`: Offers advanced Oracle capabilities for chunking documents according to different requirements. 

    * `OraLlamaVS`: Used for storing, indexing, and querying vector embeddings. 

For more information about how to use this integration for building end-to-end RAG pipelines, see [Oracle AI Vector Search: End-to-End Pipeline with Document Processing](https://github.com/run-llama/llama_index/blob/main/docs/docs/examples/cookbooks/oracleai_demo.ipynb). 




Benefits

The Oracle AI Vector Search integration with LlamaIndex provides a powerful foundation for developing sophisticated AI applications that can leverage both structured and unstructured data within the Oracle ecosystem. 

By integrating Oracle AI Vector Search with LlamaIndex, developers can: 

  * Leverage Oracle Database's enterprise features such as scalability, security, and transactions. 

  * Combine semantic search with relational queries in one single system. 

  * Utilize Oracle's optimized vector operations for efficient similarity search. 

  * Build AI-powered applications using familiar SQL and PL/SQL interfaces. 




**Parent topic:** [Use Retrieval Augmented Generation to Complement LLMs](use-retrieval-augmented-generation-complement-llms.md)
