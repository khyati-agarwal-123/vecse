##  SEARCH {#GUID-A386BDB0-35D0-41E1-8F41-49AEBEC13BFC} 

Use the ` DBMS_HYBRID_VECTOR.SEARCH ` PL/SQL function to run textual queries, vector similarity queries, or hybrid queries against hybrid vector indexes. 

Purpose 

To search by vectors and keywords. This function lets you perform the following tasks: 

  * Facilitate a combined (hybrid) query of textual documents and vectorized chunks: 

You can query a hybrid vector index in multiple vector and keyword search combinations called search modes, as described in [ Understand Hybrid Search ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-310D2298-90F4-4AFE-AF03-F3B81E55F84C) . This API accepts a JSON specification for all query parameters. 

  * Fuse and reorder the search results: 

The search results of a hybrid query are fused into a unified result set as ` CLOB ` using the specified fusion set operator, and reordered by a combined score using the specified scoring algorithm. 

  * Run a default query for a simplified search experience: 

The minimum input parameters required are the ` hybrid_index_name ` and ` search_text ` . The same text string is used to query against a vectorized chunk index and a text document index. 




Syntax 
    
    
    ```
    DBMS_HYBRID_VECTOR.SEARCH(
       json(
         '{  "hybrid_index_name":  "",                  
             "search_text"      :  "",                   
             "search_fusion"    :  "INTERSECT \| UNION \| TEXT_ONLY \| VECTOR_ONLY \| MINUS_TEXT \| MINUS_VECTOR",      
             "search_scorer"    :  "RRF \| RSF",    
             "vector":
               {
                "search_text"    :  "",            
                "search_vector"  :  "[]",           
                "search_mode"    :  "DOCUMENT \| CHUNK",               
                "aggregator"     :  "MAX \| AVG \| MEDIAN \| BONUSMAX \| WINAVG \| ADJBOOST \| MAXAVGMED",   
                "score_weight"   :   ,
                "rank_penalty"   :         
               },
             "text":
               {
                "contains"       :  "",               
                "score_weight"   :   ,
                "rank_penalty"   :       
               },
             "return":
               {
                "topN"           :   "",                               
                "values"         :    ["rowid \| score \| vector_score \| text_score \| vector_rank \| text_rank \| chunk_text \| chunk_id"],         
                "format"         :   "JSON \| XML"                              
               }
         }'
      )
    )
    ```

> **note:** This API supports two constructs of search. One where you specify a single ` search_text ` field for both semantic search and keyword search (default setting). Another where you specify separate ` search_text ` and ` contains ` query fields using ` vector ` and ` text ` sub-elements for semantic search and keyword search, respectively. You cannot use both of these search constructs in one query. 

hybrid_index_name 

Specify the name of the hybrid vector index to use. 

For information on how to create a hybrid vector index if not already created, see [ Manage Hybrid Vector Indexes ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-F2493927-23F8-4231-862B-6EDFA5A12299) . 

search_text 

Specify a search text string (your query input) for both semantic search and keyword search. 

The same text string is used for a keyword query on document text index (by converting the ` search_text ` into a ` CONTAINS ACCUM ` operator syntax) and a semantic query on vectorized chunk index (by vectorizing or embedding the ` search_text ` for a ` VECTOR_DISTANCE ` search). 

For example: 
    
    
    ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json('{ "hybrid_index_name" : "my_hybrid_idx",
                "search_text"       : "C, Python"
              }'))
    from dual;
    ```

search_fusion 

Specify a fusion sort operator to define what you want to retain from the combined set of keyword-and-semantic search results. 

> **note:** This search fusion operation is applicable only to non-pure hybrid search cases. Vector-only and text-only searches do not fuse any results. 

Parameter  |  Description   
---|---  
` INTERSECT ` (default)  |  Returns only the rows that are common to both text search results and vector search results.  Score condition: ` text_score > 0 AND vector_score > 0 `  
` UNION ` |  Combines all distinct rows from both text search results and vector search results.  Score condition: ` text_score > 0 OR vector_score > 0 `  
` TEXT_ONLY ` |  Returns all distinct rows from text search results plus the ones that are common to both text search results and vector search results. Thus, the fused results contain the text search results that appear in text search, including those that appear in both.  Score condition: ` text_score > 0 `  
` VECTOR_ONLY ` |  Returns all distinct rows from vector search results plus the ones that are common to both text search results and vector search results. Thus, the fused results contain the vector search results that appear in vector search, including those that appear in both.  Score condition: ` vector_score > 0 `  
` MINUS_TEXT ` |  Returns all distinct rows from vector search results minus the ones that are common to both text search results and vector search results. Thus, the fused results contain the vector search results that appear in vector search, excluding those that appear in both.  Score condition: ` text_score = 0 `  
` MINUS_VECTOR ` |  Returns all distinct rows from text search results minus the ones that are common to both text search results and vector search results. Thus, the fused results contain the text search results that appear in text search, excluding those that appear in both.  Score condition: ` vector_score = 0 `  
  
For example: 
    
    
    ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json('{ "hybrid_index_name"     : "my_hybrid_idx",
                "search_fusion"         : "UNION",
                "vector":
                        { "search_text" : "leadership experience" },
                 "text":
                        { "contains"    : "C and Python" }
              }'))
    from dual;
    ```

search_scorer 

Specify a method to evaluate the combined "fusion" search scores from both keyword and semantic search results. 

  * ` RSF ` (default) to use the Relative Score Fusion (RSF) algorithm 

  * ` RRF ` to use the Reciprocal Rank Fusion (RRF) algorithm 




For a deeper understanding of how these algorithms work in hybrid search modes, see [ Understand Hybrid Search ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-310D2298-90F4-4AFE-AF03-F3B81E55F84C) . 

For example: 

With a single search text string for hybrid search: 
    
    
    ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json(
          '{ "hybrid_index_name" : "my_hybrid_idx",
             "search_text"       : "C, Python",
             "search_scorer"     : "rsf"
          }'))
    from dual;
    ```

With separate vector and text search strings: 
    
    
    ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json(
          '{ "hybrid_index_name" : "my_hybrid_idx",
             "search_scorer"     : "rsf",
             "vector":
              { "search_text"    : "leadership experience" },
             "text":
              { "contains"       : "C and Python" }
          }'))
    from dual;
    ```

vector 

Specify query parameters for semantic search against the vector index part of your hybrid vector index: 

  * ` search_text ` : Search text string (query text). 

This string is converted into a query vector (embedding), and is used in a ` VECTOR_DISTANCE ` query to search against the vectorized chunk index. 

For example: 
    
        ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json('{ "hybrid_index_name" : "my_hybrid_idx",
                "vector":
                        { "search_text" : "C, Python }
              }'))
    from dual;
    ```

  * ` search_vector ` : Vector embedding (query vector). 

This embedding is directly used in a ` VECTOR_DISTANCE ` query to search against the vectorized chunk index. 

> **note:** The vector embedding that you pass here must be generated using the same embedding model used for semantic search by the specified hybrid vector index. 

For example: 
    
        ```
    dbms_hybrid_vector.search(
        json('{ "hybrid_index_name" : "my_hybrid_idx",
             "vector":
                      {
                       "search_vector" : vector_serialize(
                                                vector_embedding(doc_model
                                                            using
                                                            "C, Python, Database"
                                                            as data)
                                                        RETURNING CLOB)
                      }}')) 
    from dual;
    ```

  * ` search_mode ` : Document or chunk search mode in which you want to query the hybrid vector index: 

Parameter  |  Description   
---|---  
` DOCUMENT ` (default)  |  Returns document-level results. In document mode, the result of your search is a list of document IDs from the base table corresponding to the list of best documents identified.   
` CHUNK ` |  Returns chunk-level results. In chunk mode, the result of your search is a list of chunk identifiers and associated document IDs from the base table corresponding to the list of best chunks identified, regardless of whether the chunks come from the same document or different documents.  The content from these chunk texts can be used as input for LLMs to formulate responses.   
  
For example, semantic search in chunk mode: 
    
        ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json(
          '{ "hybrid_index_name" : "my_hybrid_idx",
             "vector":
              {
                 "search_text"   : "leadership experience",
                 "search_mode"   : "CHUNK"
              }
          }'))
    from dual;
    ```

  * ` aggregator ` : Aggregate function to apply for ranking the vector scores for each document in ` DOCUMENT SEARCH_MODE ` . 

Parameter  |  Description   
---|---  
` MAX ` (default)  |  Standard database aggregate function that selects the top chunk score as the result score.   
` AVG ` |  Standard database aggregate function that sums the chunk scores and divides by the count.   
` MEDIAN ` |  Standard database aggregate function that computes the middle value or an interpolated value of the sorted scores.   
` BONUSAVG ` |  This function combines the maximum chunk score with the remainder multiplied by the average score of the other top scores.   
` WINAVG ` |  This function computes the maximum average of the rolling window (of size ` windowSize ` ) of chunk scores.   
` ADJBOOST ` |  This function computes the average "boosted" chunk score. The chunk scores are boosted with the ` BOOSTFACTOR ` multiplied by average score of the surrounding chunk's scores (if they exist).   
` MAXAVGMED ` |  This function computes a weighted sum of the ` MAX ` , ` AVGN ` , and ` MEDN ` values.   
  
For example: 
    
        ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json(
          '{ "hybrid_index_name" : "my_hybrid_idx",
             "vector":
              {
                 "search_text"   : "leadership experience",
                 "search_mode"   : "DOCUMENT",
                 "aggregator"    : "AVG"
              }
          }'))
    from dual;
    ```

  * ` score_weight ` : Relative weight (degree of importance or preference) to assign to the semantic ` VECTOR_DISTANCE ` query. This value is used when combining the results of RSF ranking. 

Value: Any positive integer greater than ` 0 ` (zero) 

Default: ` 10 ` (implies 10 times more importance to vector query than text query) 

For example: 
    
        ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json(
          '{ "hybrid_index_name" : "my_hybrid_idx",
             "vector":
              {
                 "search_text"   : "leadership experience",
                 "search_mode"   : "DOCUMENT",
                 "aggregator"    : "MAX",
                 "score_weight"  : 5
              }
          }'))
    from dual;
    ```

  * ` rank_penalty ` : Penalty (denominator in RRF, represented as ` 1/(rank+penality ` ) to assign to vector query. This can help in balancing the relevance score by reducing the importance of unnecessary or repetitive words in a document. This value is used when combining the results of RRF ranking. 

Value: ` 0 ` (zero) or any positive integer 

Default: ` 1 `

For example: 
    
        ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json(
          '{ "hybrid_index_name" : "my_hybrid_idx",
             "search_scorer"     : "rrf",
             "vector":
              {
                 "search_text"   : "leadership experience",
                 "search_mode"   : "DOCUMENT",
                 "aggregator"    : "MAX",
                 "score_weight"  : 5,
                 "rank_penalty"  : 2
              }
          }'))
    from dual;
    ```




text 

Specify query parameters for keyword search against the Oracle Text index part of your hybrid vector index: 

  * ` contains ` : Search text string (query text). 

This string is converted into an Oracle Text ` CONTAINS ` query operator syntax for keyword search. 

You can use ` CONTAINS ` query operators to specify query expressions for full-text search, such as OR ( ` | ` ), AND ( ` & ` ), STEM ( ` $ ` ), MINUS ( ` - ` ), and so on. For a complete list of all such operators to use, see [ *Oracle Text Reference*  ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-6410B783-FC9A-4C99-B3AF-9E0349AA43D1) . 

For example: 

With a text contains string for pure keyword search: 
    
        ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json('{ "hybrid_index_name" : "my_hybrid_idx",
                "text":
                       { "contains" : "C and Python" }
              }'))
    from dual;
    ```

With separate search texts using ` vector ` and ` text ` sub-elements for hybrid search. One search text or a vector embedding to run a ` VECTOR_DISTANCE ` query for semantic search. A second search text to run a ` CONTAINS ` query for keyword search. This query conducts two separate keyword and semantic queries, where keyword scores and semantic scores are combined: 
    
        ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json('{ "hybrid_index_name" : "my_hybrid_idx",
                "vector":
                        { "search_text" : "leadership experience" },
                 "text":
                        { "contains" : "C and Python" }
              }'))
    from dual;
    ```

  * ` score_weight ` : Relative weight (degree of importance or preference) to assign to the text ` CONTAINS ` query. This value is used when combining the results of RSF ranking. 

Value: Any positive integer greater than ` 0 ` (zero) 

Default: ` 1 ` (implies neutral weight) 

For example: 
    
        ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json(
          '{ "hybrid_index_name" : "my_hybrid_idx",
             "text":
              {
                 "contains"      : "C and Python",
                 "score_weight"  : 1
              }
          }'))
    from dual;
    ```

  * ` rank_penalty ` : Penalty (denominator in RRF, represented as ` 1/(rank+penality ` ) to assign to keyword query. 

This can help in balancing the relevance score by reducing the importance of unnecessary or repetitive words in a document. This value is used when combining the results of RRF ranking. 

Value: ` 0 ` (zero) or any positive integer 

Default: ` 5 `

For example: 
    
        ```
    DBMS_HYBRID_VECTOR.SEARCH(
        json(
          '{ "hybrid_index_name" : "my_hybrid_idx",
             "text":
              {
                 "contains"      : "C and Python",
                 "rank_penalty"  : 5
              }
          }'))
    from dual;
    ```




return 

Specify which fields to appear in the result set: 

Parameter  |  Description   
---|---  
` topN ` |  Maximum number of best-matched results to be returned  Value: Any integer greater than ` 0 ` (zero)  Default: ` 20 `  
` values ` |  Return attributes for the search results. Values for scores range between 100 (best) to 0 (worse).  <ul>

<li>
  * ` rowid ` : Row ID associated with the source document.  </li> <li>
  * ` score ` : Final score computed from keyword-and-semantic search scores.  </li> <li>
  * ` vector_score ` : Semantic score from vector search results.  </li> <li>
  * ` text_score ` : Keyword score from text search results.  </li> <li>
  * ` vector_rank ` : Ranking of chunks retrieved from semantic or ` VECTOR_DISTANCE ` search.  </li> <li>
  * ` text_rank ` : Ranking of documents retrieved from keyword or ` CONTAINS ` search.  </li> <li>
  * ` chunk_text ` : Human-readable content from each chunk.  </li> <li>
  * ` chunk_id ` : ID of each chunk text.  </li> </ul> Default: All the above return attributes are shown by default.   
` format ` |  Format of the results as:  <ul> <li>
    * ` JSON ` (default)  </li> <li>
    * ` XML ` </li> </ul>  
  
For example: 
        
                ```
        DBMS_HYBRID_VECTOR.SEARCH(
            json(
              '{ "hybrid_index_name" : "my_hybrid_idx",
                 "search_text"       : "C, Python",
                 "return":
                  {
                     "values"        : [ "rowid", "score" ],
                     "topN"          : 3,
                     "format"        : "JSON"
                  }
              }'))
        from dual;
        ```

Complete Example With All Query Parameters 

The following example shows a hybrid search query that performs separate text and vector searches against ` my_hybrid_idx ` . This query specifies the ` search_text ` for vector search using the ` vector_distance ` function as ` prioritize teamwork and leadership experience ` and the keyword for text search using the ` contains ` operator as ` C and Python ` . The search mode is ` DOCUMENT ` to return the search results as topN documents. 
        
                ```
        select json_Serialize(
          DBMS_HYBRID_VECTOR.SEARCH(
            json(
              '{ "hybrid_index_name" : "my_hybrid_idx",
                 "search_fusion"     : "INTERSECT",
                 "search_scorer"     : "rsf",
                 "vector":
                  {
                     "search_text"   : "prioritize teamwork and leadership experience",
                     "search_mode"   : "DOCUMENT",
                     "score_weight"  : 10,
                     "rank_penalty"  : 1,
                     "aggregator"    : "MAX"
                  },
                 "text":
                  {
                     "contains"      : "C and Python",
                     "score_weight"  : 1,
                     "rank_penalty"  : 5
                  },
                 "return":
                  {
                     "format"        : "JSON",
                     "topN"          : 3,
                     "values"        : [ "rowid", "score", "vector_score",
                                         "text_score", "vector_rank",
                                         "text_rank", "chunk_text", "chunk_id" ]
                  }
              }'
            )
          ) pretty)
        from dual;
        ```

The top 3 rows are ordered by relevance, with higher scores indicating a better match. All the return attributes are shown by default: 
        
                ```
        [
          {
            "rowid"         : "AAAR9jAABAAAQeaAAA",
            "score"         : 58.64,
            "vector_score"  : 61,
            "text_score"    : 35,
            "vector_rank"   : 1,
            "text_rank"     : 2,
            "chunk_text"    : "Candidate 1: C Master. Optimizes low-level system (i.e. Database)
                               performance with C. Strong leadership skills in guiding teams to 
                               deliver complex projects.",
            "chunk_id"      : "1"
          },
          {
            "rowid"         : "AAAR9jAABAAAQeaAAB",
            "score"         : 56.86,
            "vector_score"  : 55.75,
            "text_score"    : 68,
            "vector_rank"   : 3,
            "text_rank"     : 1,
            "chunk_text"    : "Candidate 3: Full-Stack Developer. Skilled in Database, C, HTML,
                               JavaScript, and Python with experience in building responsive web 
                               applications. Thrives in collaborative team environments.",
            "chunk_id"      : "1"
          },
          {
            "rowid"         : "AAAR9jAABAAAQeaAAD",
            "score"         : 51.67,
            "vector_score"  : 56.64,
            "text_score"    : 2,
            "vector_rank"   : 2,
            "text_rank"     : 3,
            "chunk_text"    : "Candidate 2: Database Administrator (DBA). Maintains and secures
                               enterprise database (Oracle, MySql, SQL Server). Passionate about 
                               data integrity and optimization. Strong mentor for junior DBA(s).",
            "chunk_id"      : "1"
          }
        ]
        ```

End-to-end example  : 

To see how to create a hybrid vector index and explore all types of queries against the index, see [ Query Hybrid Vector Indexes End-to-End Example ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-4E340969-349E-4E46-AD98-AC2A0AEDB773) . 

**Related Topics**

      * [ Perform Hybrid Search ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-531F9E71-2CEF-46A8-AA53-7C161E801E3A)

**Parent topic:** [ DBMS_HYBRID_VECTOR ](dbms_hybrid_vector-vecse.md)
