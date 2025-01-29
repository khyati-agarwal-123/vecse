This diagram explains that as DMLs accumulate in the shared journal, exact searches for deleted and inserted vectors in the
            on-disk journal see their performance deteriorating. To minimize this impact, a full repopulation of the index is automatically
            triggered. The decision to repopulate the HNSW index in the background is based on the default threshold, representing a certain
            percentage of the number of DMLs run against the index. In addition, each time the HNSW index is created or repopulated, a
            Full Checkpoint of the newly created graph is created on disk.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

