This diagram shows an end-to-end indexing pipeline of a hybrid vector index. In the center, there is a datastore called MY_DS
            datastore. This datastore includes a document table DOCS that contains IDs and corresponding document names or file names
            of binary documents such as PDF, Word, or Excel. A simple CREATE HYBRID VECTOR INDEX statement is shown at the top.

On the left of the datastore, the indexing pipeline passes the documents through these stages:

Stage 1: Filter for conversion of binary documents to plain text.

Stage 2: Tokenizer for tokenization of data for keyword search.

Stage 3: Indexing Engine for the creation of a keyword index with secondary tables like $I.

On the right of the datastore, the indexing pipeline passes the documents through these stages:

Stage 1: Filter for conversion of binary documents to plain text.

Stage 2: Vectorizer for chunking and embedding generation for vector search.

Stage 3: Indexing Engine for the creation of HNSW vector index with secondary tables like $VR.

[Copyright © 2023, 2025, Oracle and/or its affiliates.](../../../dcommon/html/cpyr.htm)

