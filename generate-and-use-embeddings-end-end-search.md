## Generate and Use Embeddings for an End-to-End Search {#GUID-68EA9930-1623-4D27-ACE2-5F0760A3930D}

First generate vector embeddings from textual content by using a vector embedding model stored in the database, and then populate and query a vector index. At query time, you also vectorize the query criteria on the fly.

This example covers the entire workflow of Oracle AI Vector Search (as explained in [Understand the Stages of Data Transformations](understand-stages-data-transformations.md#GUID-D6F1E7B6-5642-46A2-B84D-B8E37E4C353E)). If you are not yet familiar with the concepts beyond generating embeddings (such as creating and querying vector indexes), review the remaining sections before running this scenario. 

To run an end-to-end similarity search workflow by accessing a vector embedding model stored in the database:

  1. Start SQL*Plus and connect to Oracle Database as a local user:
    1. Log in to SQL*Plus as the `SYS` user, connecting as `SYSDBA`:
```
        conn sys/password as sysdba
```
```
        CREATE TABLESPACE tbs1
        DATAFILE 'tbs5.dbf' SIZE 20G AUTOEXTEND ON
        EXTENT MANAGEMENT LOCAL
        SEGMENT SPACE MANAGEMENT AUTO;
```
```
        SET ECHO ON
        SET FEEDBACK 1
        SET NUMWIDTH 10
        SET LINESIZE 80
        SET TRIMSPOOL ON
        SET TAB OFF
        SET PAGESIZE 10000
        SET LONG 10000
```
        

    2. Create a local user (`docuser`) and grant necessary privileges:
```
        drop user docuser cascade;
```
```
        create user docuser identified by docuser DEFAULT TABLESPACE tbs1 quota unlimited on tbs1;
```
```
        grant DB_DEVELOPER_ROLE to docuser;
```
        

    3. Create a local directory (`VEC_DUMP`) to store your input data and model files. Grant necessary privileges:
```
        create or replace directory VEC_DUMP as '/my_local_dir/';
```
```
        grant read, write on directory VEC_DUMP to docuser;
        
        commit;
```
        

    4. Connect as the local user (`docuser`):
```
        conn docuser/password
```
        

  2. Create a relational table (`documentation_tab`) and store your textual content in it.
```
    drop table documentation_tab purge;
```
```
    create table documentation_tab (id number, text clob);
```
```
    insert into documentation_tab values (1,
    'Analytics empowers business analysts and consumers with modern, AI-powered, self-service analytics capabilities for data preparation, visualization, enterprise reporting, augmented analysis, and natural language processing.
    Oracle Analytics Cloud is a scalable and secure public cloud service that provides capabilities to explore and perform collaborative analytics for you, your workgroup, and your enterprise.
    
    Oracle Analytics Cloud is available on Oracle Cloud Infrastructure Gen 2 in several regions in North America, EMEA, APAC, and LAD when you subscribe through Universal Credits. You can subscribe to Professional Edition or Enterprise Edition.');
    
    insert into documentation_tab values (3,
    'Generative AI Data Science is a fully managed and serverless platform for data science teams to build, train, and manage machine learning models in the Oracle Cloud Infrastructure.');
    
    insert into documentation_tab values (4,
    'Language allows you to perform sophisticated text analysis at scale. Using the pretrained and custom models, you can process unstructured text to extract insights without data science expertise.
    Pretrained models include sentiment analysis, key phrase extraction, text classification, and named entity recognition. You can also train custom models for named entity recognition and text
    classification with domain specific datasets. Additionally, you can translate text across numerous languages.');
    
    insert into documentation_tab values (5,
    'When you work with Oracle Cloud Infrastructure, one of the first steps is to set up a virtual cloud network (VCN) for your cloud resources. This topic gives you an overview of Oracle Cloud
    Infrastructure Networking components and typical scenarios for using a VCN. A virtual, private network that you set up in Oracle data centers. It closely resembles a traditional network, with
    firewall rules and specific types of communication gateways that you can choose to use. A VCN resides in a single Oracle Cloud Infrastructure region and covers one or more CIDR blocks
    (IPv4 and IPv6, if enabled). See Allowed VCN Size and Address Ranges. The terms virtual cloud network, VCN, and cloud network are used interchangeably in this documentation.
    For more information, see VCNs and Subnets.');
    
    insert into documentation_tab values (6,
    'NetSuite banking offers several processing options to accurately track your income. You can record deposits to your bank accounts to capture customer payments and other monies received in the
    course of doing business. For a deposit, you can select payments received for existing transactions, add funds not related to transaction payments, and record any cash received back from the bank.');
    
    commit;
```
    

  3. Load your embedding model by calling the `load_onnx_model` procedure.
```
    EXECUTE dbms_vector.drop_onnx_model(model_name => 'doc_model', force => true);
```
```
    EXECUTE dbms_vector.load_onnx_model(
    'VEC_DUMP',
    'my_embedding_model.onnx',
    'doc_model',
    json('{"function" : "embedding", "embeddingOutput" : "embedding" , "input": {"input": ["DATA"]}}')
    );
```
    

In this example, the procedure loads an ONNX model file, named `my_embedding_model.onnx` from the `VEC_DUMP` directory, into the database as `doc_model`. You must replace `my_embedding_model.onnx` with an ONNX export of your embedding model and `doc_model` with the name under which the imported model is stored in the database. 

> **note:** If you do not have an embedding model in ONNX format, then perform the steps listed in [ONNX Pipeline Models : Text Embedding](onnx-pipeline-models-text-embedding.md#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6). 

  4. Create a relational table (`doc_chunks`) to store unstructured data chunks and associated vector embeddings, by using `doc_model`. 
```
    create table doc_chunks as (
    SELECT d.id id,
    row_number() over (partition by d.id order by d.id) chunk_id,
    vc.chunk_offset chunk_offset,
    vc.chunk_length chunk_length,
    vc.chunk_text chunk,
    vector_embedding(doc_model using vc.chunk_text as data) vector
    FROM documentation_tab d,
    vector_chunks(d.text by words max 100 overlap 10 split RECURSIVELY) vc
    );
```
    

The `CREATE TABLE` statement reads the text from the `DOCUMENTATION_TAB` table, first applies the `VECTOR_CHUNKS` SQL function to split the text into chunks based on the specified chunking parameters, and then applies the `VECTOR_EMBEDDING` SQL function to generate corresponding vector embedding on each resulting chunk text. 

  5. Explore the `doc_chunks` table by selecting rows from it to see the chunked output. 
```
    desc doc_chunks;
    set linesize 100
    set long 1000
    col id for 999
    col chunk_id for 99999
    col chunk_offset for 99999
    col chunk_length for 99999
    col chunk for a30
    col vector for a100
```
```
    select id, chunk_id, chunk_offset, chunk_length, chunk from doc_chunks;
```
    

The chunking output returns a set of seven chunks, which are split recursively, that is, using the `BLANKLINE`, `NEWLINE`, `SPACE`, `NONE` sequence. Note that Document 5 produces two chunks when the maximum word limit of `100` is reached. 

You can see that the first chunk ends at a blank line. The text from the first chunk overlaps onto the second chunk, that is, 10 words (including comma and period; underlined below) are overlapping. Similarly, there is an overlap of 10 (also underlined below) between the fifth and sixth chunks. 
```
    ID CHUNK_ID CHUNK_OFFSET CHUNK_LENGTH CHUNK
    ---- -------- ------------ ------------ ------------------------------
    1	 1	     1	   418  Analytics empowers business an
    alysts and consumers with mode
    rn, AI-powered, self-service a
    nalytics capabilities for data
    preparation, visualization, e
    nterprise reporting, augmented
    analysis, and natural languag
    e processing.
    Oracle Analytics Cloud is
    a scalable and secure public
    cloud service that provides ca
    pabilities to explore and perf
    orm collaborative analytics for
    you, your workgroup, and your
    enterprise.
    
    1	 2	     373	 291 for you, your workgroup, and
    your enterprise.
    Oracle Analytics Cloud is
    available on Oracle Cloud Inf
    rastructure Gen 2 in several r
    egions in North America, EMEA,
    APAC, and LAD when you subscr
    ibe through Universal Credits.
    You can subscribe to Professi
    onal Edition or Enterprise Edi
    tion.
    
    3	 1		1	 180  Generative AI Data Science is
    a fully managed and serverless
    platform for data science tea
    ms to build, train, and manage
    machine learning models in th
    e Oracle Cloud Infrastructure.
    
    4	 1		1	 505  Language allows you to perform
    sophisticated text analysis a
    t scale. Using the pretrained
    and custom models, you can pro
    cess unstructured text to extr
    act insights without data scie
    nce expertise.
    Pretrained models include
    sentiment analysis, key phras
    e extraction, text classificat
    ion, and named entity recognit
    ion. You can also train custom
    models for named entity recog
    nition and text
    classification with domai
    n specific datasets. Additiona
    lly, you can translate text ac
    ross numerous languages.
    
    5	 1		1	 386  When you work with Oracle Clou
    d Infrastructure, one of the f
    irst steps is to set up a virt
    ual cloud network (VCN) for yo
    ur cloud resources. This topic
    gives you an overview of Orac
    le Cloud
    Infrastructure Networking
    components and typical scenar
    ios for using a VCN. A virtual
    , private network that you set
    up in Oracle data centers. It
    closely resembles a tradition
    al network, with
    
    5	 2	     329	 474  centers. It closely resembles
    a traditional network, with
    firewall rules and specif
    ic types of communication gate
    ways that you can choose to us
    e. A VCN resides in a single O
    racle Cloud Infrastructure reg
    ion and covers one or more CID
    R blocks
    (IPv4 and IPv6, if enable
    d). See Allowed VCN Size and A
    ddress Ranges. The terms virtu
    al cloud network, VCN, and clo
    ud network are used interchang
    eably in this documentation.
    For more information, see
    VCNs and Subnets.
    
    6	 1		1	 393  NetSuite banking offers severa
    l processing options to accura
    tely track your income. You ca
    n record deposits to your bank
    accounts to capture customer
    payments and other monies rece
    ived in the
    course of doing business.
    For a deposit, you can select
    payments received for existin
    g transactions, add funds not
    related to transaction payment
    s, and record any cash receive
    d back from the bank.
    
    7 rows selected.
```
    

  6. Explore the first vector result by selecting rows from the `doc_chunks` table to see the embedding output. 
```
    select vector from doc_chunks where rownum <= 1;
```
    

An excerpt from the output is as follows:
```
    [1.18813422E-002,2.53968383E-003,-5.33896387E-002,1.46877998E-003,5.77209815E-002,-1.58939194E-002,3
    .12595293E-002,-1.13087103E-001,8.5138239E-002,1.10731693E-002,3.70671228E-002,4.03710492E-002,1.503
    95066E-001,3.31836529E-002,-1.98343433E-002,6.16453104E-002,4.2827677E-002,-4.02921103E-002,-7.84291
    551E-002,-4.79201972E-002,-5.06678E-002,-1.36317732E-002,-3.7761624E-003,-2.3332756E-002,1.42400926E
    -002,-1.11553416E-001,-3.70503664E-002,-2.60722954E-002,-1.2074843E-002,-3.55089158E-002,-1.03518805
    E-002,-7.05051869E-002,5.63110895E-002,4.79055084E-002,-1.46315445E-003,8.83129537E-002,5.12795225E-
    002,7.5858552E-003,-4.13030013E-002,-5.2099824E-002,5.75958602E-002,3.72097567E-002,6.11167103E-002,
    ,-1.23207876E-003,-5.46219759E-003,3.04734893E-002,1.80617068E-002,-2.85708476E-002,-1.01670986E-002
    ,6.49402961E-002,-9.76506807E-003,6.15146831E-002,5.27246818E-002,7.44994432E-002,-5.86469211E-002,8
    .84285953E-004,2.77456306E-002,1.99283361E-002,2.37570312E-002,2.33389344E-002,-4.07911092E-002,-7.6
    1070028E-002,1.23929314E-001,6.65794984E-002,-6.15389943E-002,2.62510721E-002,-2.48490628E-002]
```
    

  7. Create an index on top of the `doc_chunks` table's `vector` column.
```
    create vector index vidx on doc_chunks (vector)
    organization neighbor partitions
    with target accuracy 95
    distance EUCLIDEAN parameters (
    type IVF,
    neighbor partitions 2);
```
    

  8. Run queries using the vector index.

     * Query about Machine Learning:
```
        select id, vector_distance(
        vector,
        vector_embedding(doc_model using 'machine learning models' as data),
        EUCLIDEAN) results
        FROM doc_chunks order by results;
```
```
        ID    RESULTS
        ---- ----------
        3 1.074E+000
        4 1.086E+000
        5 1.212E+000
        5 1.296E+000
        1 1.304E+000
        6 1.309E+000
        1 1.365E+000
        
        7 rows selected.
```
        

     * Query about Generative AI:
```
        select id, vector_distance(
        vector,
        vector_embedding(doc_model using 'gen ai' as data),
        EUCLIDEAN) results
        FROM doc_chunks order by results;
```
```
        ID	RESULTS
        ---- ----------
        4 1.271E+000
        3 1.297E+000
        1 1.309E+000
        5  1.32E+000
        1 1.352E+000
        5 1.388E+000
        6 1.424E+000
        
        7 rows selected.
```
        

     * Query about Networks:
```
        select id, vector_distance(
        vector,
        vector_embedding(doc_model using 'computing networks' as data),
        MANHATTAN) results
        FROM doc_chunks order by results;
```
```
        ID	RESULTS
        ---- ----------
        5 1.387E+001
        5 1.441E+001
        3 1.636E+001
        1 1.707E+001
        4 1.758E+001
        1 1.795E+001
        6 1.902E+001
        
        7 rows selected.
```
        

     * Query about Banking:
```
        select id, vector_distance(
        vector,
        vector_embedding(doc_model using 'banking, money' as data),
        MANHATTAN) results
        FROM doc_chunks order by results;
```
```
        ID	RESULTS
        ---- ----------
        6 1.363E+001
        1 1.969E+001
        5 1.978E+001
        5 1.997E+001
        3 1.999E+001
        1 2.058E+001
        4 2.079E+001
        
        7 rows selected.
```
        




**Related Topics**

  * [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504)



**Parent topic:** [Perform Chunking With Embedding](perform-chunking-embedding.md)
