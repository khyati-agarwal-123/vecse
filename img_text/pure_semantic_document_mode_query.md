This diagram shows a simple DBMS_HYBRID_VECTOR.SEARCH statement to run a pure semantic search query in document mode. You can see that a similarity search is run on all the vectors
            and extracts the top k ones.  at most. These k vectors (at most) are then grouped by document IDs and for each identified
            document, the semantic score of each associated vector found for that document is used to compute the average (in this example)
            of these scores for that particular document. The top 10 documents (at most) with the highest averages are returned.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

