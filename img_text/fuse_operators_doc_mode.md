This diagram shows that first the keyword search doc IDs and vector search doc IDs are submitted to a UNION ALL operation. Then, the possible SEARCH_FUSION operators are shown, such as Union (which combines both the results), Intersect (which combines the common results), Text_Only (which combines the text search results plus the common text-vector search results), Vector_Only (which combines the vector search results plus the common text-vector search results), Minus_Vector (which includes all the text results minus the common text-vector search results), and Minus_Text (which includes all vector results minus all the common vector-text search results).

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

