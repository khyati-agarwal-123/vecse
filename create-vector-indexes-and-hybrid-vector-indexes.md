## 6Create Vector Indexes and Hybrid Vector Indexes {#GUID-8AF956F3-D951-4968-9B79-A6E180E87456}

Depending on whether to use similarity search or hybrid search, you can create either a vector index or a hybrid vector index.

**Vector indexes** are a class of specialized indexing data structures that are designed to accelerate similarity searches using high-dimensional vectors. They use techniques such as clustering, partitioning, and neighbor graphs to group vectors representing similar items, which drastically reduces the search space, thereby making the search process extremely efficient. Create vector indexes on your vector embeddings to use these indexes for running similarity searches over huge vector spaces. 

**Hybrid vector indexes** leverage the existing Oracle Text indexing data structures and vector indexing data structures. Such indexes can provide more relevant search results by integrating the keyword matching capabilities of text search with the semantic precision of vector search. Create hybrid vector indexes directly on your input data or on your vector embeddings to use these indexes for performing a combination of full-text search and similarity search. 

  * [Size the Vector Pool](size-vector-pool.md)  
To allow vector index creation, you must enable a new memory area stored in the SGA called the **Vector Pool**. 
  * [Manage the Different Categories of Vector Indexes](manage-different-categories-vector-indexes.md)  

  * [Manage Hybrid Vector Indexes](manage-hybrid-vector-indexes.md)  
Learn how to manage a hybrid vector index, which is a single index for searching by *similarity* and *keywords*, to enhance the accuracy of your search results. 
  * [Vector Indexes in a Globally Distributed Database](vector-indexes-gdd.md)  
Inverted File Flat (IVF) index, Hierarchical Navigable Small World (HNSW) index, and Hybrid Vector Index (HVI) are supported on sharded tables in a distributed database; however there are some considerations. 


