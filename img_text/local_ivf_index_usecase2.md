This diagram illustrates a local IVF index that contains two internal tables, Centroids and CentroidPartitions:

The Centroids table is list-partitioned by the base table subpartition ids. Each partition in the Centroids table contains
            all centroid vectors (C1, C2, C3) found in the corresponding base table subpartition (P1SP1 and P1SP2, P2SP1 and P2SP2, and
            finally P3SP1 and P3SP2).

The CentroidPartitions table is list-partitioned by base table subpartition id, and is further subpartitioned by centroid
            id. That is, P1SP1C1, P1SP1C2, and P1SP1C3 for subpartition P1SP1, P1SP2C1, P1SP2C2, and P1SP2C3 for subpartition P1SP2, P2SP1C1,
            P2SP1C2, and P2SP1C3 for subpartition P2SP1, P2SP2C1, P2SP2C2, and P2SP2C3 for subpartition P2SP2, P3SP1C1, P3SP1C2, and P3SP1C3
            for subpartition P3SP1, and finally P3SP2C1, P3SP2C2, and P3SP2C3 for subpartition P3SP2.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

