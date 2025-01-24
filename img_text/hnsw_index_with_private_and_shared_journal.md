This diagram explains that the current HNSW graph in memory as well as the query's private journal and shared journal are
            used to determine transactionally-consistent set of deleted and inserted vectors.

This consists of identifying the exact lists of deleted vectors from the journals, running a approximate top-K search through
            the current version of the HNSW index by augmenting the filter to ignore deleted vectors, running an exact top-k search of
            newly inserted vectors in the journals, and merging the results of the two searches.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

