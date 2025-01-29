This diagram illustrates how in a Hierarchical Navigable Small World (HNSW) graph, starting from the top layer, the search
            in one layer starts at the entry vector. Then for each node, if there is a neighbor that is closer to the query vector than
            the current node, it jumps to that neighbor. The algorithm keeps doing this until it finds a local minimum for the query vector.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

