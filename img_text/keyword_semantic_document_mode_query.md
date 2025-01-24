This diagram outlines how keyword scores and semantic scores are combined for a Keyword-and-Semantic search in document mode.
            It shows that first, both search results are added using a UNION ALL operation. Then you specify the SEARCH_FUSION operation. Then final scoring is computed using the defined SEARCH_SCORER algorithm such as Reciprocal Rank Fusion (RRF) or Relative Score Fusion (RSF). Finally, the topN document identifiers are returned at most.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

