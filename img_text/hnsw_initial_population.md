This diagram shows the process of RAC-wide HNSW index population in an Oracle RAC environment. After the first HNSW graph
            is created, all other RAC instances are informed to start their own HNSW In-Memory Graph creation concurrently through an
            HNSW duplication operation. The duplication mechanism is using the HNSW ROWID-to-VID mapping table on disk.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

