## Perform Multi-Vector Similarity Search {#GUID-F5ACF1FB-87C8-4051-872D-63E408480EC8}

Another major use-case of vector search is multi-vector search. Multi-vector search is typically associated with a multi-document search, where documents are split into chunks that are individually embedded into vectors.

A multi-vector search consists of retrieving top-K vector matches using grouping criteria known as partitions based on the documents' characteristics. This ability to score documents based on the similarity of their chunks to a query vector being searched is facilitated in SQL using the partitioned row limiting clause.

With multi-vector search, it is easier to write SQL statements to answer the following type of question:

  * If they exist, what are the four best matching sentences found in the three best matching paragraphs of the two best matching books?



For example, imagine if each book in your database is organized into paragraphs containing sentences which have vector embedding representations, then you can answer the previous question using a single SQL statement such as:
```
    SELECT bookId, paragraphId, sentence
    FROM books
    ORDER BY vector_distance(sentence_embedding, :sentence_query_vector)
    FETCH FIRST 2 PARTITIONS BY bookId, 3 PARTITIONS BY paragraphId, 4 ROWS ONLY;
```
    

You can also use an approximate similarity search instead of an exact similarity search as shown in the following example:
```
    SELECT bookId, paragraphId, sentence
    FROM books
    ORDER BY vector_distance(sentence_embedding, :sentence_query_vector)
    FETCH APPROXIMATE FIRST 2 PARTITIONS BY bookId, 3 PARTITIONS BY paragraphId, 4 ROWS ONLY
    WITH TARGET ACCURACY 90;
```
    

> **note:** All the rows returned are ordered by `VECTOR_DISTANCE()` and not grouped by the partition clause. 

Semantically, the previous SQL statement is interpreted as:

  * Sort all records in the books table in descending order of the vector distance between the sentences and the query vector.
  * For each record in this order, check its `bookId` and `paragraphId`. This record is produced if the following three conditions are met: 
    1. Its `bookId` is one of the first two distinct `bookId` in the sorted order. 
    2. Its `paragraphId` is one of the first three distinct `paragraphId` in the sorted order within the same `bookId`. 
    3. Its record is one of the first four records within the same `bookId` and `paragraphId` combination. 
  * Otherwise, this record is filtered out.



Multi-vector similarity search is not just for documents and can be used to answer the following questions too:

  * Return the top K closest matching photos but ensure that they are photos of different people.
  * Find the top K songs with two or more audio segments that best match this sound snippet.



> **note:** 

  * This partition row-limiting clause extension is a generic extension of the SQL language. It does not have to apply just to vector searches.
  * Multi-vector search with the partitioning row-limit clause does not use vector indexes.



> **note:** See Also: 

[*Oracle Database SQL Language Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SQLRF-GUID-CFA006CA-6FF1-4972-821E-6996142A51C6) for the full syntax of the `ROW_LIMITING_CLAUSE`

**Parent topic:** [Query Data With Similarity and Hybrid Searches](query-data-similarity-and-hybrid-searches.md)
