## Convert File to Text to Chunks to Embeddings Within Oracle Database {#GUID-10D0A76C-2DAE-42BE-A332-3EEF6D91D701}

First convert a PDF file to text, split the text into chunks, and then create vector embeddings on each chunk by accessing a vector embedding model stored in the database.

You can run parallel or step-by-step transformations like this for standalone applications where you want to review, inspect, and accordingly amend results at each stage and then proceed further.

Here, you use a set of functions from the `DBMS_VECTOR_CHAIN` package, such as `UTL_TO_TEXT`, `UTL_TO_CHUNKS`, and `UTL_TO_EMBEDDINGS`. 

To generate embeddings using a vector embedding model stored in the database, through step-by-step transformation chains:

  1. **Connect as a local user and prepare your data dump directory**.
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
        

  2. **Convert file to text**.
    1. Create a relational table (`documentation_tab`) and store your PDF document (*Oracle Database Concepts*) in it:
```
        drop table documentation_tab purge;
```
```
        CREATE TABLE documentation_tab (id number, data blob);
        
        INSERT INTO documentation_tab values(1, to_blob(bfilename('VEC_DUMP', 'database-concepts23ai.pdf')));
        
        commit;
```
```
        SELECT dbms_lob.getlength(t.data) from documentation_tab t;
```
        

    2. Call `UTL_TO_TEXT` to convert the PDF document into text format:
```
        SELECT dbms_vector_chain.utl_to_text(dt.data) from documentation_tab dt;
```
        

An excerpt from the output is as follows:
```
    DBMS_VECTOR_CHAIN.UTL_TO_TEXT(DT.DATA)
    --------------------------------------------------------------------------
    Database Concepts
    23ai
    Oracle Database Database Concepts, 23ai
    
    This software and related documentation are provided under a license agreement containing restrictions on
    use and disclosure and are protected by intellectual property laws. Except as expressly permitted in your
    license agreement or allowed by law, you may not use, copy, reproduce, translate
    , broadcast, modify, license, transmit, distribute, exhibit, perform, publish, or display any part, in any for
    m, or by any means. Reverse engineering, disassembly, or decompilation of this software, unless required by
    law for interoperability, is prohibited.
    
    Contents
    
    Preface
    Audience
    xxiii
    Documentation Accessibility
    xxiii
    Related Documentation
    xxiv
    Conventions
    xxiv
    1
    Introduction to Oracle Database
    About Relational Databases
    1-1
    Database Management System (DBMS)
    1-2
    Relational Model
    1-2
    Relational Database Management System (RDBMS)
    1-3
    Brief History of Oracle Database
    1-3
    Schema Objects
    1-5
    Tables
    1-5
    Indexes
    1-6
    Data Access
    
    1 row selected.
```
    

  3. **Convert text to chunks**.
    1. Call `UTL_TO_CHUNKS` to chunk the text document:

Here, you create the chunks using the default chunking parameters.
```
        SELECT ct.*
        from documentation_tab dt,
        dbms_vector_chain.utl_to_chunks(dbms_vector_chain.utl_to_text(dt.data)) ct;
```
        

An excerpt from the output is as follows:
```
        {"chunk_id":1,"chunk_offset":1508024,"chunk_length":579,"chunk_data":"Inventory
        \n\n\n\nAnalysis \n\n\n\nReporting \n\n\n\nMining\n\n\n\nSummary \n\n\n\nData
        \n\n\n\nRaw Data\n\n\n\nMetadata\n\n\n\nSee Also:\n\n\n\nOracle Database Data
        Warehousing Guide to learn about transformation \n\n\n\nmechanisms\n\n\n\nOvervie
        w of Extraction, Transformation, and Loading (ETL) \n\n\n\nThe process of extrac
        ting data from source systems and bringing it into the warehouse is \n\n\n\ncomm
        only called ETL: extraction, transformation, and loading. ETL refers to a broad
        process \n\n\n\nrather than three well-defined steps.\n\n\n\nIn a typical scenar
        io, data from one or more operational systems is extracted and then"}
        
        {"chunk_id":2,"chunk_offset":1508603,"chunk_length":607,"chunk_data":"physica
        lly transported to the target system or an intermediate system for processing. \
        n\n\n\nDepending on the method of transportation, some transformations can occur
        during this \n\n\n\nprocess. For example, a SQL statement that directly accesse
        s a remote target through a \n\n\n\ngateway can concatenate two columns as part
        of the \n\n\n\nSELECT\n\n\n\nstatement. \n\n\n\nOracle Database is not itself an
        ETL tool. However, Oracle Database provides a rich set of \n\n\n\ncapabilities
        usable by ETL tools and customized ETL solutions. ETL capabilities provided by \
        n\n\n\nOracle Database include:\n\n\n\n? \n\n\n\nTransportable tablespaces"}
        
        3728 rows selected.
```
        

Notice the extra spaces and newline characters (`\n\n`) in the chunked output. This is because normalization is not applied by default. As shown in the next step, you can apply custom chunking specifications, such as `normalize` to omit duplicate characters, extra spaces, or newline characters from the output. You can further refine your chunks by applying other chunking specifications, such as split conditions or maximum size limits. 

    2. Apply the following JSON parameters to use normalization and some of the custom chunking specifications (described in [Explore Chunking Techniques and Examples](explore-chunking-techniques-and-examples.md#GUID-BFDE8B0A-6302-472C-AD8B-1BEB9AA2CB87)):
```
        SELECT ct.*
        from documentation_tab dt,
        dbms_vector_chain.utl_to_chunks(dbms_vector_chain.utl_to_text(
        dt.data),
        JSON('{
        "by" : "words",
        "max" : "100",
        "overlap" : "0",
        "split" : "recursively",
        "language" : "american",
        "normalize" : "all"
        }')) ct;
```
        

The output may now appear as:
```
        {"chunk_id":2536,"chunk_offset":1372527,"chunk_length":633,"chunk_data":"The dat
        abase maps granules to parallel execution servers at execution time. When a para
        llel execution server finishes reading the rows corresponding to a granule, and
        when granules remain, it obtains another granule from the query coordinator. This
        operation continues until the table has been read. The execution servers send
        results back to the coordinator, which assembles the pieces into the desired full
        table scan. Oracle Database VLDB and Partitioning Guide to learn how to use
        parallel execution. Oracle Database Data Warehousing Guide to learn about
        recommended"}
        
        {"chunk_id":2537,"chunk_offset":1373160,"chunk_length":701,"chunk_data":"initial
        ization parameters for parallelism\n\nChapter 18\n\nOverview of Background Proce
        sses\n\nApplication and Oracle Net Services Architecture\n\nThis chapter defines
        application architecture and describes how an Oracle database and database appli
        cations work in a distributed processing environment. This material applies to
        almost every type of Oracle Database environment. Overview of Oracle Application
        Architecture In the context of this chapter, application architecture refers to
        the computing environment in which a database application connects to an Oracle
        database."}
        
        3728 rows selected.
```
        

The chunking results contain: 
       * `chunk_id`: Chunk ID for each chunk 

       * `chunk_offset`: Original position of each chunk in the source document, relative to the start of document (which has a position of `1`) 

       * `chunk_length`: Character length of each chunk 

       * `chunk_data`: Text pieces from each chunk 

  4. **Convert chunks to embeddings**.
    1. Load your embedding model into Oracle Database by calling the `load_onnx_model` procedure.
```
        EXECUTE dbms_vector.drop_onnx_model(model_name => 'doc_model', force => true);
```
```
        EXECUTE dbms_vector.load_onnx_model('VEC_DUMP', 'my_embedding_model.onnx', 'doc_model', JSON('{"function" : "embedding", "embeddingOutput" : "embedding" , "input": {"input": ["DATA"]}}'));
```
        

In this example, the procedure loads an ONNX model file, named `my_embedding_model.onnx` from the `VEC_DUMP` directory, into the database as `doc_model`. You must replace `my_embedding_model.onnx` with an ONNX export of your embedding model and `doc_model` with the name under which the imported model is stored in the database. 

> **note:** If you do not have an embedding model in ONNX format, then perform the steps listed in [ONNX Pipeline Models : Text Embedding](onnx-pipeline-models-text-embedding.md#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6). 

    2. Call `UTL_TO_EMBEDDINGS` to generate a set of vector embeddings corresponding to the chunks:
```
        var embed_params clob;
        exec :embed_params := '{"provider":"database", "model":"doc_model"}';
        
        SELECT et.* from
        documentation_tab dt,
        dbms_vector_chain.utl_to_embeddings(
        dbms_vector_chain.utl_to_chunks(dbms_vector_chain.utl_to_text(dt.data)),
        json(:embed_params)) et;
```
        

An excerpt from the output is as follows:
```
    {"embed_id":"1","embed_data":"Introduction to Oracle Database\n\n\nThis
    chapter provides an overview of Oracle Database. Every organization has informat
    ion that it must store and manage to meet its requirements. For example, a corpo
    ration must collect and maintain human resources records for its employees. About
    Relational Databases and Schema Objects\n\n\nData Access tables with parent keys,
    base, upgrade, UROWID data type, user global area (UGA), user program interface,
    ","embed_vector":"[0.111119926,0.0423980951,-0.00929224491,-0.0352411047,-0.0144
    591287,0.0277361721,0.183199733,-0.0245029964,-0.137614027,0.0730137378,0.017934
    6036,0.0788726509,0.0176453814,0.100403085,-0.0518687107,-0.0152645027,0.0283792
    187,-0.114087239,0.0139923804,0.0747490972,-0.181839675,-0.130034953,0.101207718
    ,0.117135495,-0.0682030097,-0.217743069,0.0613380745,0.0150767341,0.0361393057,-
    0.113082513,-0.0550440662,-0.000983044971,-0.00719357422,0.1590323,-0.0220414512
    ,-0.0723528489,-0.0126240514,-0.175765082,0.168952227,0.0466451086,-0.12136507,-
    0.0442310236,0.0139067639,0.054659389,-0.29653421,-0.0988782048,0.0794349238,-0.
    0758788213,0.0152856084,-0.0260562375,0.0652966872,-0.0782724097,-0.0226081386,0
    .0909011662,-0.184569761,0.159565002,-0.15350005,-0.0108382348,0.101788878,1.919
    59683E-002,2.54665539E-002,2.50248201E-002,5.29858321E-002,1.42359538E-002,5.655
    82886E-002,3.41602638E-002,3.18607911E-002,-3.07250433E-002,-3.60006578E-002,-3.
    26940455E-002,-5.13980947E-002,-9.18597169E-003,-2.40122043E-002,2.15246622E-002
    ,-3.89301814E-002,1.09825116E-002,-8.59739035E-002,-3.34327705E-002,-6.52310252E
    -002,2.46418975E-002,6.27725571E-003,6.54156879E-002,-2.97986511E-003,-1.485541E
    -003,-9.00155635E-003]"}
    
    {"embed_id":2,"embed_data":"1-18 Database, 19-6 write-ahead,18-17 Learn session
    memory in a large pool. The multitenant architecture enables an Oracle database
    to function as a multitenant container database (CDB). XStream,20-38\nZero Data
    Loss Recovery Appliance See Recovery Appliance zone maps, An application contai
    ner consists of exactly one application root, and PDBs plugged in to this root.
    Index-21\n D:20231208125114-08'00' D:20231208125114-08'00' D:20231208125114-08'0
    0' 19-6","embed_vector":"[6.30229115E-002,6.80943206E-002,-6.94655553E-002,-2.58
    157589E-002,-1.89648587E-002,-9.02655348E-002,1.97774544E-002,-9.39233322E-003,-
    5.06882742E-002,2.0078931E-002,-1.28898735E-003,-4.10606936E-002,2.09831214E-003
    ,-4.53372523E-002,-7.09890276E-002,5.38906306E-002,-5.81014939E-002,-1.3959175E-
    004,-1.08725717E-002,-3.79145369E-002,-4.39973129E-003,3.25602405E-002,6.5887302
    2E-002,-4.27212603E-002,-3.00925318E-002,3.07144262E-002,-2.26370787E-004,-4.623
    15865E-002,1.11807801E-001,7.36674219E-002,-1.61244173E-003,7.35205635E-002,4.16
    726843E-002,-5.08309156E-002,-3.55720241E-003,4.49763797E-003,5.03803678E-002,2.
    32542045E-002,-2.58533042E-002,9.06257033E-002,8.49585086E-002,8.65213498E-002,5
    .84013164E-002,-7.72946924E-002,6.65430725E-002,-1.64568517E-002,3.23978886E-002
    ,2.24988302E-003,3.02566844E-003,-2.43405364E-002,9.75424573E-002,4.14630538E-00
    3,1.89351812E-002,-1.10467218E-001,-1.24333188E-001,-2.36738548E-002,7.54277706E
    -002,-1.64660662E-002,-1.38906585E-002,3.42438952E-003,-1.88432514E-005,-2.47511
    379E-002,-3.42802797E-003,3.23110656E-003,4.24311385E-002,6.59448802E-002,-3.311
    67318E-002,-5.14010936E-002,2.38897409E-002,-9.00154635E-002]"}
    
    3728 rows selected.
```
    

The embedding results contain: 
     * `embed_id`: ID number of each vector embedding 

     * `embed_data`: Input text that is transformed into embeddings 

     * `embed_vector`: Generated vector representations 




**Related Topics**

  * [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12)
  * [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504)



**Parent topic:** [Perform Chunking With Embedding](perform-chunking-embedding.md)
