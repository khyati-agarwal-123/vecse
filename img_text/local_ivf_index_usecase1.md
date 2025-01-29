This diagram illustrates a local IVF index that contains two internal tables, Centroids and CentroidPartitions:

The Centroids table is list-partitioned by the base table partition ids, and is thus equi-partioned with the base table. Each
            partition (P1, P2, and P3) contains a list of corresponding identified centroid vectors and associated ids (C1, C2, and C3).

The CentroidPartitions table is list-partitioned by base table partition id and list-subpartitioned by centroid id. This table
            is also equi-partitioned with the base table, and each subpartition contains the base table vectors closely related (cluster)
            to the corresponding centroid id for that subpartition. That is, P1_C1, P1_C2, and P1_C3 for partition P1, P2_C1, P2_C2, and
            P2_C3 for partition P2, and similarly P3_C1, P3_C2, and P3_C3 for partition P3.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

