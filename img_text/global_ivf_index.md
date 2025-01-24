In this diagram, it is explained that a global IVF vector index itself is composed of two tables.

The Centroids table contains a list of identified centroid vectors and associated ids, C1, C2, and C3.

The CentroidPartitions table is list-partitioned on the centroid ids. Each partition contains the base table vectors closely
            related (cluster) to the corresponding centroid id for that partition, P1(C1), P2(C2), and P3(C3).

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

