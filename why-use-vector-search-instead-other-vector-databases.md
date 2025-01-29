## Why Use Oracle AI Vector Search? {#GUID-7516630E-2533-473A-95DA-0020347E150A}

One of the biggest benefits of Oracle AI Vector Search is that semantic search on unstructured data can be combined with relational search on business data in one single system.

This is not only powerful but also significantly more effective because you don't need to add a specialized vector database, eliminating the pain of data fragmentation between multiple systems.

For example, suppose you use an application that allows you to find a house that is similar to a picture you took of one you like that is located in your preferred area for a certain budget. Finding a good match in this case requires combining a semantic picture search with searches on business data.

With Oracle AI Vector Search, you can create the following table:
```
    CREATE TABLE house_for_sale (house_id     NUMBER,
    price        NUMBER,
    city         VARCHAR2(400),
    house_photo  BLOB,
    house_vector VECTOR);
```
    

The following sections of this guide describe in detail the meaning of the `VECTOR` data type and how to load data in this column data type. 

With that table, you can run the following query to answer your basic question:
```
    SELECT house_photo, city, price
    FROM   house_for_sale
    WHERE  price <= :input_price AND
    city  = :input_city
    ORDER BY VECTOR_DISTANCE(house_vector, :input_vector);
```
    

Later sections of this guide describe in detail the meaning of the `VECTOR_DISTANCE` function. This query is just to show you how simple it is to combine a vector embedding similarity search with relation predicates. 

In conjunction with Oracle Database 23ai, Oracle Exadata System Software release 24.1.0 introduces AI Smart Scan, a collection of Exadata-specific optimizations capable of improving the performance of various AI vector query operations by orders of magnitude.

AI Smart Scan automatically accelerates Oracle Database 23ai AI Vector Search with optimizations that deliver low-latency parallelized scans across massive volumes of vector data. AI Smart Scan processes vector data at memory speed, leveraging ultra-fast Exadata RDMA Memory (XRMEM) and Exadata Smart Flash Cache in the Exadata storage servers, and performs vector distance computations and top-K filtering at the data source, avoiding unnecessary network data transfer and database server processing.

**Parent topic:** [Overview](overview-node.md)
