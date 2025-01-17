[Previous](search.md) Next JavaScript must be enabled to correctly display this content 

## Glossary {#topic}

### dimension {#GUID-FD84D4FD-2FBE-4276-9FBB-E12F559EC7E7}

A dimension refers to an array element of a vector.

**Parent topic:** [Glossary](glossary.md)

### dimension format {#GUID-EF108278-4FF0-430D-9D1F-CC2FE99ABF5E}

The dimensions of a vector can be represented using numbers of varying types and precisions, called the dimension format. The dimensions supported by vector embeddings in Oracle Database are `INT8` (1-byte signed integer), `FLOAT32` (4-byte single-precision floating point number), and `FLOAT64` (8-byte double-precision floating point number). All dimensions of a vector must have the same dimension format. 

**Parent topic:** [Glossary](glossary.md)

### distance metric {#GUID-055CD8F9-DF2F-456E-9012-EB48CAD772BA}

Distance metric refers to the mathematical function used to compute distance between vectors. Popular distance metrics supported by Oracle AI Vector search include Euclidean distance, Cosine distance, and Manhattan distance, among others.

**Parent topic:** [Glossary](glossary.md)

### embedding model {#GUID-C5B08EDB-7039-4DED-A844-13640FC82F3B}

Embedding models are Machine Learning algorithms that are trained to capture semantic information of unstructured data and represent it as vectors in multidimensional space. Different embedding models exist for different types of unstructured data, for example, BERT for Text data, ResNet-50 for Image data, and so on.

**Parent topic:** [Glossary](glossary.md)

### hybrid search {#GUID-B3F3BFFC-1C8E-43BE-BE08-26A082028815}

Hybrid search is an advanced information retrieval technique that lets you search documents by both *keywords* and *vectors*. Hybrid searches are run against hybrid vector indexes by querying it in various search modes. By integrating traditional keyword-based text search with vector-based similarity search, you can improve the overall search experience and provide users with more relevant information. 

**Parent topic:** [Glossary](glossary.md)

### hybrid vector index {#GUID-0E88D5B6-8D71-4CEA-AF45-266344B95EBA}

A hybrid vector index is a class of specialized Domain Index that combines the existing Oracle Text search indexes and Oracle AI Vector Search vector indexes into one unified structure. A single index contains both textual and vector fields for a document, enabling you to perform a combination of keyword-based text search and vector-based similarity search simultaneously.

**Parent topic:** [Glossary](glossary.md)

### large language model {#GUID-F63E6554-C179-46A6-8092-F1A724E5F848}

Large language models (LLMs) are advanced Machine Learning models designed to understand, process, and generate natural language for rich human interaction. They are typically built using deep learning algorithms and are pretrained on vast amounts of data. Popular examples include Open AI’s GPT-4, Cohere’s Command R+, and Meta’s LLaMa 3.

**Parent topic:** [Glossary](glossary.md)

### multi-vector {#GUID-EDE07C45-2CD0-4A01-B9D1-5D4F62CAB3DD}

Multi-vector refers to a scenario where multiple vectors correspond to a single entity. For example, a large text document can be chunked into paragraphs and every paragraph can be embedded into a separate vector. A similarity search query could retrieve matching documents based on the most similar paragraph (closest vectors) per document to a given query vector. Oracle AI Vector Search has the option of Partitioned Row Limiting Fetch syntax to enable efficient multi-vector searches.

**Parent topic:** [Glossary](glossary.md)

### neighbor graph {#GUID-740CD1A8-B9EF-4F0C-B636-78B911DA0B2A}

A neighbor graph is a graph-based data structure used in vector indexes. For example, a Hierarchical Navigable Small World (HNSW) vector index leverages a multilayer neighbor graph index. In a neighbor graph, each vertex of the graph represents a vector in the data set, and edges are created between vertices representing similar vectors.

**Parent topic:** [Glossary](glossary.md)

### query accuracy {#GUID-4A6CD4ED-3457-4B6A-BF1C-C857EB39DC0B}

Query accuracy is an intuitive indicator of the quality of an approximate query result obtained from a vector index search. Consider a query vector, for which an exact search, that searches through all vectors in the data set, returns Top 5 matches as `{ID1, ID3, ID5, ID7, ID9}`, and an approximate vector index search returns Top 5 matches as `{ID1, ID3, ID5, ID9, ID10}`. Since the approximate result has 4 out of 5 correct matches, the query accuracy is 80%. 

**Parent topic:** [Glossary](glossary.md)

### query vector {#GUID-A8372703-723A-4FFD-9457-A01A6C9A83BB}

Query vector refers to the vector embedding representing the item for which the user wants to find similar items using [similarity search](glossary.md#GUID-1AA2FD8B-F838-4940-9F98-3BAFABEDADF6). For example, while searching for movies similar to a user’s favorite movie, the vector embedding representing the user’s favorite movie is the query vector. 

**Parent topic:** [Glossary](glossary.md)

### Retrieval-Augmented Generation {#GUID-2FED3DAE-A2C2-41D3-B015-4E56411017D5}

Retrieval-Augmented Generation (RAG) is a popular technique for enhancing the accuracy of responses generated by [Large Language Models](glossary.md#GUID-F63E6554-C179-46A6-8092-F1A724E5F848) by augmenting the user-provided prompt with relevant, up-to-date, enterprise-specific content retrieved using AI Vector Search. Applications, such as Chat Assistants, built using RAG are often more accurate, reliable, and cost-effective. 

**Parent topic:** [Glossary](glossary.md)

### similarity search {#GUID-1AA2FD8B-F838-4940-9F98-3BAFABEDADF6}

Similarity Search is a common operation in information retrieval to find items in a data set that are similar to a user-provided query item. For example, finding movies similar to a user’s favorite movie is an example of similarity search. Vectors can enable efficient similarity searches by leveraging the property that the mathematical distance between vectors is a proxy for similarity, as in, the more similar two items are, the smaller the distance between the vectors.

**Parent topic:** [Glossary](glossary.md)

### vector {#GUID-460ED712-0539-456B-8638-C536CC80CD8E}

A vector is a mathematical entity that has a magnitude and a direction. It is typically represented as an array of numbers, which are coordinates that define its position in a multidimensional space. 

**Parent topic:** [Glossary](glossary.md)

### vector distance {#GUID-4A4CB8D2-9D89-4BCD-99B6-E01D4DA212C1}

Vector distance refers to the mathematical distance between two vectors in a multidimensional space. The vector distance between similar items is smaller than the vector distance between dissimilar items. Vector distance is meaningful only if the vectors being compared are generated by the same embedding model.

**Parent topic:** [Glossary](glossary.md)

### vector embedding {#GUID-E7AC75C4-5A8B-4CB0-8A6A-53D961C0BB36}

A vector embedding is a numerical representation of text, image, audio, or video data that encodes the semantic content of the data, and not the underlying words or pixels. The terms *vector* and *vector embedding* are often used interchangeably in AI Vector Search. 

**Parent topic:** [Glossary](glossary.md)

### vector index {#GUID-42EBEEEE-EDC0-4C94-A81A-33A7F5CB163A}

Vector indexes are a class of specialized indexing data structures that are designed to accelerate similarity searches using high-dimensional vectors. They use techniques such as clustering, partitioning, and neighbor graphs to group vectors representing similar items, which drastically reduces the search space, thereby making the search process extremely efficient. Unlike traditional databases indexes, vector indexes enable approximate similarity searches, which allow users to trade off query accuracy for query performance to better suit application requirements.

**Parent topic:** [Glossary](glossary.md)

### Vector Memory Pool {#GUID-5E2FB75D-3C52-4617-8AF1-9696CD0A7661}

Vector Memory Pool is a region of the System Global Area (SGA) that is dedicated to storing In-Memory Neighbor Graph Vector Indexes (HNSW index), as well as metadata for Neighbor Partition Vector Indexes. It can be specified by using the `VECTOR_MEMORY_SIZE` parameter. 

**Parent topic:** [Glossary](glossary.md)