## Perform Hybrid Search {#GUID-531F9E71-2CEF-46A8-AA53-7C161E801E3A}

Hybrid search is an advanced information retrieval technique that lets you search documents by *keywords* and *vectors*, to achieve more relevant search results. 

  * [Understand Hybrid Search](understand-hybrid-search.md)  
With hybrid search, you can search through your documents by performing a combination of full-text queries and vector-based similarity queries, using out-of-the-box or custom scoring techniques. 
  * [Query Hybrid Vector Indexes End-to-End Example](query-hybrid-vector-indexes-end-end-example.md)  
In this example, you can see an end-to-end hybrid search workflow. First, you run the `CREATE HYBRID VECTOR INDEX` SQL statement that prepares, chunks, embeds, stores, and indexes your input data. You then perform vector search alongside keyword search using the `DBMS_HYBRID_VECTOR.SEARCH` PL/SQL query API. 



**Parent topic:** [Query Data With Similarity and Hybrid Searches](query-data-similarity-and-hybrid-searches.md)
