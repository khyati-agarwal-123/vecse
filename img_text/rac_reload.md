This diagram explains the reload mechanism in a RAC environment. Here, you can see a Disk structure containing the ROWID-to-VID
            Mapping Table, Shared Journal, and Full Checkpoint. Four stages are explained as follows:

Stage 1 shows that the in-memory HNSW graph and the ROWID-to-VID mapping table are created on disks at index creation time.
            It also shows that the VECTOR_INDEX_NEIGHBOR_GRAPH_RELOAD instance parameter is set to RESTART, which indicates that automatic reload should take place.

Stage 2 shows that, if enabled, a full checkpoint is also created on disks.

Stage 3 shows that the shared journal structure is also created on disks.

Stage 4 shows that if the VECTOR_INDEX_NEIGHBOR_GRAPH_RELOAD instance parameter is set to RESTART and if a full checkpoint exists and is not deemed too old compared to the current SCN, then it is used by the starting instance
            to create its HNSW graph in memory. Because a full checkpoint is optional, if it does not exist at instance restart or if
            it exists but is deemed too old compared to the current SCN, then the ROWID-to-VID mapping table is used to reload the graph
            in memory.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

