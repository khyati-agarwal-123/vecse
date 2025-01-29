## Vectorize Relational Tables Using OML Feature Extraction Algorithms {#GUID-DB1E43F3-699D-4739-AB33-34E66F0B3726}

This example shows you how to use OML's Feature Extraction algorithms in conjunction with the `VECTOR_EMBEDDING()` operator to vectorize sets of relational data, build similarity indexes, and perform similarity searches on the resulting vectors. 

Feature Extraction algorithms help in extracting the most informative features/columns from the data and aim to reduce the dimensionality of large data sets by identifying the principal components that capture the most variance in the data. This reduction simplifies the data set while retaining the most important information, making it easier to analyze correlations and redundancies in the data.

The Principal Component Analysis (PCA) algorithm, a widely used dimensionality reduction technique in machine learning, is used in this tutorial.

> **note:** 

This example uses customer bank marketing data available at <https://archive.ics.uci.edu/dataset/222/bank+marketing>. 

The relational data table includes a mix of numeric and categorical columns. It has more than 4000 records.
```
    SELECT column_name, data_type
    FROM user_tab_columns
    WHERE table_name = 'BANK'
    ORDER BY data_type, column_name;
    
    
    COLUMN_NAME          DATA_TYPE
    -------------------- --------------------
    AGE                  NUMBER
    CAMPAIGN             NUMBER
    CONS_CONF_IDX        NUMBER
    CONS_PRICE_IDX       NUMBER
    DURATION             NUMBER
    EMP_VAR_RATE         NUMBER
    EURIBOR3M            NUMBER
    ID                   NUMBER
    NR_EMPLOYED          NUMBER
    PDAYS                NUMBER
    PREVIOUS             NUMBER
    CONTACT              VARCHAR2
    CREDIT_DEFAULT       VARCHAR2
    DAY_OF_WEEK          VARCHAR2
    EDUCATION            VARCHAR2
    HOUSING              VARCHAR2
    JOB                  VARCHAR2
    LOAN                 VARCHAR2
    MARITAL              VARCHAR2
    MONTH                VARCHAR2
    POUTCOME             VARCHAR2
    Y                    VARCHAR2
```
    

To perform a similarity search, you need to vectorize the relational data. To do so, you can first use the OML Feature Extraction algorithm to project the data onto a more compact numeric space. In this example, you configure the SVD algorithm to perform a Principal Component Analysis (PCA) projection of the original data table. The number of features/columns (5 in this case) is specified in the setting table. The input number determines the number of principal features or columns that will be retained after the dimensionality reduction process. Each of these columns represent a direction in the feature space along which the data varies the most.

Because you need to use the `DBMS_DATA_MINING` package to create the model, you need the `CREATE MINING MODEL` privilege in addition to the other privileges relevant to vector indexes and similarity search. For more information about the `CREATE MINING MODEL` privilege, see [*Oracle Machine Learning for SQL User’s Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=DMPRG-GUID-C837421E-F237-46F0-A26C-C1FFDDA47B1E). 

  1. Create a setting table, insert values, and then create a model.

Use the `DBMS_DATA_MINING` package to create a model, using `mod_sett` as the setting table: 
```
    CREATE TABLE mod_sett(
    setting_name  VARCHAR2(30),
    setting_value VARCHAR2(30)
    );
    
    
    BEGIN
    INSERT INTO mod_sett (setting_name, setting_value) VALUES
    (dbms_data_mining.algo_name, dbms_data_mining.algo_singular_value_decomp);
    INSERT INTO mod_sett (setting_name, setting_value) VALUES
    (dbms_data_mining.prep_auto, dbms_data_mining.prep_auto_on);
    INSERT INTO mod_sett (setting_name, setting_value) VALUES
    (dbms_data_mining.**svds_scoring_mode**, dbms_data_mining.svds_scoring_pca);
    INSERT INTO mod_sett (setting_name, setting_value) VALUES
    (dbms_data_mining.**feat_num_features**, 5);
    commit;
    END;
    /
    
    
    BEGIN
    DBMS_DATA_MINING.CREATE_MODEL(
    model_name          => 'pcamod',
    mining_function     => dbms_data_mining.feature_extraction,
    data_table_name     => 'bank',
    case_id_column_name => 'id',
    settings_table_name => 'mod_sett');
    END;
    /
```
    

  2. Use the `VECTOR_EMBEDDING()` function to output the SVD projection results as vectors.

The dimension of the vector column is the same as the number of features in the PCA model and the value of the vector represents the PCA projection results of the original row data.

> **note:** `USING *` results in the use of all the relevant features present in the input row. 
```
    SELECT id, vector_embedding(pcamod USING *) embedding
    FROM bank
    WHERE id=10000;
    
    ID      EMBEDDING
    --------------     --------------------------------------------------
    10000      [-2.3551013972411354E+002,2.8160084506788273E+001,
    5.2821278275005774E+001,-1.8960922352439308E-002,
    -2.5441143639048378E+000]
```
    

  3. Create a table to hold the vector output results for all the data records.

This represents a vectorization of your original relational data for the top-5 most important columns.
```
    CREATE TABLE pca_output AS
    (SELECT id, **vector_embedding**(pcamod USING *) embedding
    FROM bank);
```
    

  4. Build a vector index based on the vector table.

For example, you can build an IVF index with cosine distance:
```
    CREATE VECTOR INDEX my_ivf_idx ON pca_output(embedding) ORGANIZATION
    NEIGHBOR PARTITIONS
    DISTANCE COSINE WITH TARGET ACCURACY 95;
```
    

  5. Perform a similarity search using the `my_ivf_idx` index.

In this example, you search for the top-3 results that are closest to `id=10000` based on the cosine distance and join the vector table with the original bank table to retrieve the most impactful attributes from the original table. To identify the most impactful columns for this row, use the `FEATURE_DETAILS()` function. 
```
    SELECT feature_details(pcamod, 5 USING *) features
    FROM bank
    WHERE id=10000;
    
    
    FEATURES
    -----------------------------------------------------------------------------------------------
```
    

Join the original data table to retrieve the most impactful information:
```
    SELECT p.id id, b.PDAYS **PDAYS**, b.EURIBOR3M **EURIBOR3M**, b.CONTACT **CONTACT**,
    b.EMP_VAR_RATE **EMP_VAR_RATE**, b.DAY_OF_WEEK **DAY_OF_WEEK**
    FROM pca_output p, bank b
    WHERE p.id <> 10000 AND p.id=b.id
    ORDER BY VECTOR_DISTANCE(embedding, (select embedding from pca_output where id=10000), COSINE)
    FETCH APPROXIMATE FIRST 3 ROWS ONLY;
    
    
    ID      PDAYS  EURIBOR3M   CONTACT EMP_VAR_RATE DAY_OF_WEEK
    ---------- ---------- ---------- --------- ------------ -----------
    9416        999      4.967 telephone          1.4         fri
    13485        999      4.963 telephone          1.4         thu
    9800        999      4.959 telephone          1.4         wed
```
    

The results of the previous query illustrate that the closest 3 records are very similar. In contrast, the distributions of these features across the data set are dispersed as shown in the following queries:
```
    SELECT avg(PDAYS) avg, stddev(PDAYS) std, min(PDAYS) min, max(PDAYS) max
    FROM bank;
    
    AVG        STD        MIN        MAX
    ---------- ---------- ---------- ----------
    962.475454 186.910907          0        999
    
    
    
    SELECT avg(EURIBOR3M) avg, stddev(EURIBOR3M) std, min(EURIBOR3M) min, max(EURIBOR3M) max
    FROM bank;
    
    AVG       STD        MIN        MAX
    ---------- --------- ---------- ----------
    
    3.62129081 1.7344474       .634      5.045
```
    

This tutorial demonstrates how you can vectorize relational data very efficiently and achieve significant compression while maintaining a high quality similarity search.

> **note:** See Also: 

[*Oracle Machine Learning for SQL Concepts*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=DMCON-GUID-EA778C1F-7819-4C31-8E49-299F57C9B709) for more information about Feature Extraction algorithms 




**Parent topic:** [Generate Embeddings](generate-embeddings.md)
