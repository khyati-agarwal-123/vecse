##  SQL Quick Start Using a BINARY Vector Generator {#GUID-28958BF9-9294-406B-B47F-C800D38F4C07} 

A set of procedures generate ` BINARY ` vectors, providing a simple way to get started with Oracle AI Vector Search without a vector embedding model. 

The included procedures allow you to randomly generate binary vectors with a specified number of dimensions and clusters. The output of the generation process is the population of tables called ` genbvec ` and ` genbvec_i ` that you can then use, for example, to experiment with similarity searches. 

The following instructions assume you already have access to a database account with sufficient privileges (minimally the ` DB_DEVELOPER_ROLE ` role). 

> **note:** Do not use the ` BINARY ` vector generator on production databases. This tutorial is made available for testing and demo purposes. 

> **note:** ` BINARY ` vectors are not currently supported in PL/SQL, thus, the ` genbvec_i ` table is used as an intermediary table using a ` VARCHAR2 ` type argument rather than ` VECTOR ` . 

> **note:** If you already have access to a third-party ` BINARY ` vector embedding model, you can perform a real text-to- ` BINARY ` -embedding transformation by calling third-party REST APIs using the Vector Utility PL/SQL package ` DBMS_VECTOR ` . For more information, refer to the example in [ Convert Text String to BINARY Embedding Outside Oracle Database ](convert-text-string-binary-embedding-oracle-database.html#GUID-425922B4-C9DA-48B6-B98F-7846D53AE0AF) . 

  1. Create the ` genbvec ` and ` genbvec_i ` tables. 
    
        ```
    DROP TABLE genbvec PURGE;
    DROP TABLE genbvec_i PURGE;
    
    CREATE TABLE genbvec (
      id NUMBER,            -- id of the generated vector
      v VECTOR(*, BINARY),  -- generated vector
      name VARCHAR2(500),   -- name for the generated vector: C1 to Cn are centroids, Cx_y is vector number y in cluster number x
      bv VARCHAR2,          -- normalized version of th generated vector
      ly NUMBER             -- random number you can use to filter out rows in addiion to similarity search on vectors
    );
    
    CREATE TABLE genbvec_i (
      id NUMBER,            -- id of the generated vector
      v VARCHAR2,           -- generated vector
      name VARCHAR2(500),   -- name for the generated vector: C1 to Cn are centroids, Cx_y is vector number y in cluster number x
      bv VARCHAR2,          -- bit version of generated vector
      ly NUMBER             -- random number you can use to filter out rows in addition to similarity search on vectors
    );
    ```

  2. Create the procedures used to generate ` BINARY ` vectors: 
    
        ```
    CREATE OR REPLACE PROCEDURE generate_random_binary_vector(
    dimensions IN NUMBER,
    result_int OUT VARCHAR2,
    result_binary OUT VARCHAR2
    ) IS
        binary_vector VARCHAR2;
        int8_value NUMBER;
        number_of_bits NUMBER;
        char_vector VARCHAR2;
    BEGIN
      -- Validate dimension is a multiple of 8
      IF MOD(dimensions, 8) != 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Number of dimensions must be a multiple of 8');
      END IF;
    
      -- Generate the random binary vector
      binary_vector := '';
      FOR i IN 1 .. dimensions LOOP
        IF DBMS_RANDOM.VALUE(0, 1) < 0.5 THEN
          binary_vector := binary_vector || '0';
        ELSE
          binary_vector := binary_vector || '1';
        END IF;
      END LOOP;
    
      -- Convert 8-bit packets to their int8 values and build the result string
      number_of_bits := dimensions/8;
      char_vector := '[';
      FOR i IN 0 .. number_of_bits - 1 LOOP
        int8_value := 0;
        FOR j IN 0 .. 7 LOOP
          int8_value := int8_value + TO_NUMBER(SUBSTR(binary_vector, i*8+j+1, 1)) * POWER(2, j);
        END LOOP;
        char_vector := char_vector || int8_value;
        IF i < number_of_bits - 1 THEN
          char_vector := char_vector || ',';
        END IF;
      END LOOP;
      char_vector := char_vector || ']';
      
      -- Return the generated vector value
      result_int := char_vector;
      result_binary := binary_vector;
    END generate_random_binary_vector;
    /
    
    ```
    
        ```
    CREATE OR REPLACE PROCEDURE generate_binary_cluster(
      centroid IN VARCHAR2,            -- a string of 1 and 0
      spread IN NUMBER,                -- Maximum Hamming distance between centroid and other vectors in the same cluster
      cluster_size IN NUMBER,          -- Number of vectors to generate in addition to the centroid
      result_binary OUT SYS_REFCURSOR,
      result_int8 OUT SYS_REFCURSOR
    ) IS
        dimension NUMBER;
        max_spread NUMBER;
        vector VARCHAR2;
        char_vector VARCHAR2;
        flip_positions DBMS_SQL.VARCHAR2_TABLE;
        random_position NUMBER;
        tresult_binary DBMS_SQL.VARCHAR2_TABLE;
        tresult_int8 DBMS_SQL.VARCHAR2_TABLE;
        binary_vector VARCHAR2;
        cluster_index NUMBER := 1;
        number_of_bits NUMBER;
        int8_value NUMBER;
    BEGIN
      -- Determine the dimension of the centroid vector
      dimension := LENGTH(centroid);
    
      -- Ensure dimension is a multiple of 8
      IF MOD(dimension, 8) != 0 THEN
        RAISE_APPLICATION_ERROR(-20001, 'Number of dimensions must be a multiple of 8');
      END IF;
    
      -- Generate the cluster of binary vectors
      WHILE cluster_index <= cluster_size LOOP
        binary_vector := centroid;
    
        -- Randomly flip bits in the centroid vector with a max of spread bits
        max_spread := TRUNC(DBMS_RANDOM.VALUE(1, spread+1));
        flip_positions.DELETE;
        FOR i IN 1 .. max_spread LOOP
          random_position := TRUNC(DBMS_RANDOM.VALUE(1, dimension+1));
          -- Ensure no duplicates
          WHILE flip_positions.EXISTS(random_position) LOOP
            random_position := TRUNC(DBMS_RANDOM.VALUE(1, dimension+1));
          END LOOP;
          flip_positions(random_position) := '1';
        END LOOP;
    
        -- Apply flips to binary vector
        FOR i IN 1 .. dimension LOOP
          IF flip_positions.EXISTS(i) THEN
            IF SUBSTR(binary_vector, i, 1) = '0' THEN
              binary_vector := SUBSTR(binary_vector, 1, i-1) || '1' || SUBSTR(binary_vector, i+1);
            ELSE
              binary_vector := SUBSTR(binary_vector, 1, i-1) || '0' || SUBSTR(binary_vector, i+1);
            END IF;
          END IF;
        END LOOP;
    
        -- Convert binary vector to int8 values
        number_of_bits := dimension/8;
        char_vector := '[';
        FOR i IN 0 .. number_of_bits-1 LOOP
          int8_value := 0;
          FOR j IN 0 .. 7 LOOP
            int8_value := int8_value + TO_NUMBER(SUBSTR(binary_vector, i*8+j+1, 1)) * POWER(2, j);
          END LOOP;
          char_vector := char_vector || int8_value;
          IF i < number_of_bits-1 THEN
            char_vector := char_vector || ',';
          END IF;
        END LOOP;
        char_vector := char_vector || ']';
    
        -- Add generated vectors to result tables
        tresult_binary(cluster_index) := binary_vector;
        tresult_int8(cluster_index) := char_vector;
    
        cluster_index := cluster_index + 1;
        END LOOP;
    
        -- Open cursor for binary result set
        OPEN result_binary FOR
          SELECT COLUMN_VALUE AS binary_vector
          FROM TABLE(tresult_binary);
        
        -- Open cursor for int8 result set
        OPEN result_int8 FOR
          SELECT COLUMN_VALUE AS int8_vector
          FROM TABLE(tresult_int8);
    
    END generate_binary_cluster;
    /
    ```
    
        ```
    CREATE OR REPLACE PROCEDURE generate_binary_vectors_i(
      num_vectors NUMBER,   -- If numbers of vector is not a multiple of num_clusters, remaining vectors are not generated
      num_clusters NUMBER,  -- Must be greater than 0
      dimensions NUMBER,    -- Must be a multiple of 8
      cluster_spread NUMBER -- Maximum Hamming distance between centroid and other vectors in the same cluster: max number of bits flipped
    ) IS
        vectors_per_cluster NUMBER;
        remaining_vectors NUMBER;
        i NUMBER := 1;
        j NUMBER := 1;
        idx NUMBER := 1;
        max_id NUMBER;
        ri VARCHAR2;
        rb VARCHAR2;
        result_binary SYS_REFCURSOR;
        result_int8 SYS_REFCURSOR;
        vb VARCHAR2;
        vi VARCHAR2;    
    BEGIN
    
      IF (num_vectors) <=0 OR (num_clusters < 1) OR (num_vectors < num_clusters) 
            OR (dimensions <= 0) OR (dimensions > 504) OR (cluster_spread <= 0) THEN
        RAISE_APPLICATION_ERROR(-20001, 'Issues with arguments provided');
      END IF;
    
      SELECT MAX(id) INTO max_id FROM genbvec_i;
      IF max_id IS NULL THEN max_id := 0;
      END IF;
    
      -- Calculate vectors per cluster
      vectors_per_cluster := TRUNC(num_vectors / num_clusters);
      remaining_vectors := num_vectors MOD num_clusters; -- remaining vectors are not generated
    
      -- Generate cluster centroids
      FOR i IN 1..num_clusters LOOP
        generate_random_binary_vector(dimensions, ri, rb);
        INSERT INTO genbvec_i VALUES (max_id + idx, ri, 'C'||i, rb, DBMS_RANDOM.VALUE(3,600000000));
        idx := idx + 1;
    
        -- Generate vectors for each cluster
        IF vectors_per_cluster > 1 THEN
          generate_binary_cluster(rb, cluster_spread, vectors_per_cluster, result_binary, result_int8);
    
          -- Output the binary result
          j:= 1;
          LOOP
            FETCH result_binary INTO vb;
            FETCH result_int8 INTO vi;
            EXIT WHEN result_binary%NOTFOUND;
            INSERT INTO genbvec_i VALUES (max_id + idx, vi, 'C'||i||'-'||j, vb, DBMS_RANDOM.VALUE(3,600000000));
            -- DBMS_OUTPUT.PUT_LINE(v_binary);
            j := j+1;
            idx := idx + 1;
          END LOOP;
          CLOSE result_binary; 
          CLOSE result_int8;
        END IF;
      END LOOP;
      COMMIT;     
    
    END generate_binary_vectors_i;
    /
    ```

  3. After you have your vector generator procedures set up, you can run the commands in this step to get started experimenting with ` BINARY ` vectors in the database. 

This example generates two clusters, each having twenty-one 32-dimension vectors (including the centroid) with a maximum spread of 3 from the centroid: 

    1. Start out by generating some ` BINARY ` vectors using the ` generate_binary_vectors_i ` procedure. The results of the generation are inserted into the intermediate table, ` genbvec_i ` . 
        
                ```
        BEGIN
          generate_binary_vectors_i(
            num_vectors  =>   40,   -- If numbers of vector is not a multiple of num_clusters, remaining vectors are not generated
            num_clusters =>    2,   -- Must be grather than 0
            dimensions   =>   32,   -- Must be a multiple of 8 and less than 504
            cluster_spread =>  3    -- Maximum Hamming distance between centroid and other vectors in the same cluster: max number of bits flipped
          );
        END;
        /
        ```

    2. The contents of table ` genbvec_i ` are then inserted into the ` genvec ` table. Note that the second argument is inserted as a ` VECTOR ` using the ` VARCHAR2 ` data in table ` genbvec_i ` . 
        
                ```
        INSERT INTO genbvec
          SELECT id, TO_VECTOR(v, *, BINARY), name, bv, ly
          FROM genbvec_i;
        ```

Run a ` SELECT ` statement to view the generated ` BINARY ` vectors. 
        
                ```
        SELECT genbvec.*
        FROM genbvec;
        ```

Example output: 
        
                ```
           ID V                  NAME     BV                                                                            LY
        _____ __________________ ________ ___________________________________ ____________________________________________
            1 [116,26,26,89]     C1       00101110010110000101100010011010        578917568.933208703514499318085552412675
            2 [116,90,26,88]     C1-1     00101110010110100101100000011010         551454868.84699441406146771734351824486
            3 [118,90,26,89]     C1-2     01101110010110100101100010011010       26190772.38704177622293243971278825843729
            4 [117,91,26,89]     C1-3     10101110110110100101100010011010        393261986.460545883788457133759653084901
            5 [116,10,26,73]     C1-4     00101110010100000101100010010010        301925154.316362642596571564320520080501
            6 [100,30,26,89]     C1-5     00100110011110000101100010011010        78561050.7368890112264123668521415628632
            7 [116,90,26,89]     C1-6     00101110010110100101100010011010       51996417.99223649834217535861296732062946
            8 [116,18,26,217]    C1-7     00101110010010000101100010011011        408346021.677330978172185055255452444713
            9 [116,26,26,25]     C1-8     00101110010110000101100010011000        349447824.903334465535022736162718561795
           10 [124,22,26,89]     C1-9     00111110011010000101100010011010         593437043.74264986759203090834922034868
           11 [116,26,24,89]     C1-10    00101110010110000001100010011010        485114745.516221794177912861724471688657
           12 [100,10,26,89]     C1-11    00100110010100000101100010011010        244950670.092234463558553033637811041326
           13 [124,26,26,89]     C1-12    00111110010110000101100010011010        174745388.558765768889261306616051006355
           14 [116,24,58,89]     C1-13    00101110000110000101110010011010        405709800.596277187511634749489897045128
        
           ID V                   NAME     BV                                                                            LY
        _____ ___________________ ________ ___________________________________ ____________________________________________
           15 [116,155,26,89]     C1-14    00101110110110010101100010011010        507266914.788785109884027456476376277487
           16 [244,26,26,89]      C1-15    00101111010110000101100010011010        352428714.346465340894239664666410602614
           17 [244,26,26,89]      C1-16    00101111010110000101100010011010        485005891.803174230138455116927959913215
           18 [116,154,58,91]     C1-17    00101110010110010101110011011010        437431910.704840936023185383410409608034
           19 [54,30,26,89]       C1-18    01101100011110000101100010011010        449799690.697624672302215117802472491652
           20 [52,26,90,25]       C1-19    00101100010110000101101010011000        150290349.317348894840369840141941321083
           21 [116,26,18,92]      C1-20    00101110010110000100100000111010        192263276.572792730726559459488963519686
           22 [180,77,206,50]     C2       00101101101100100111001101001100        103170455.327181970861020810151564589859
           23 [180,77,202,34]     C2-1     00101101101100100101001101000100        233044043.499898587071065204178044901673
           24 [181,93,206,178]    C2-2     10101101101110100111001101001101        251946589.346377512742760888510475023443
           25 [148,77,198,50]     C2-3     00101001101100100110001101001100        225891344.380226691041721938840984846054
           26 [148,77,206,114]    C2-4     00101001101100100111001101001110        443263346.536337832918234760514987156609
           27 [180,93,206,179]    C2-5     00101101101110100111001111001101       34073319.14579670145563110851320062323881
           28 [244,77,206,114]    C2-6     00101111101100100111001101001110        207219422.319930699905577280055016447806
        
           ID V                   NAME     BV                                                                            LY
        _____ ___________________ ________ ___________________________________ ____________________________________________
           29 [180,77,206,114]    C2-7     00101101101100100111001101001110       81973075.29748040239560872445873412759189
           30 [182,77,238,50]     C2-8     01101101101100100111011101001100        512013307.704982213183805366575015724013
           31 [176,77,198,54]     C2-9     00001101101100100110001101101100        182277609.137893068494410590922900801667
           32 [180,77,206,54]     C2-10    00101101101100100111001101101100        228615086.475398587514626583018651573622
           33 [252,73,206,50]     C2-11    00111111100100100111001101001100         502944248.56385003421137245506443464308
           34 [180,13,206,50]     C2-12    00101101101100000111001101001100         384676219.84554824073067498597228195579
           35 [52,77,207,178]     C2-13    00101100101100101111001101001101        195748553.822210758344290416638357025314
           36 [180,221,206,50]    C2-14    00101101101110110111001101001100        318502358.998212510303198039256491220943
           37 [182,77,206,54]     C2-15    01101101101100100111001101101100        335010280.028749889723514492929709631314
           38 [181,77,238,34]     C2-16    10101101101100100111011101000100        130621137.002392411800662146077683792226
           39 [180,77,204,50]     C2-17    00101101101100100011001101001100       44349671.27813632454990679483352172837714
           40 [182,77,206,50]     C2-18    01101101101100100111001101001100        249325607.576808570532467366276102484952
           41 [244,77,78,50]      C2-19    00101111101100100111001001001100        239623832.541073481316101186242859770744
           42 [188,205,206,50]    C2-20    00111101101100110111001101001100         306473398.64810863268599576461087059874
        
        
        42 rows selected.
        ```

    3. Perform a similarity search on the ` BINARY ` vectors. 
        
                ```
        SELECT name, v, bv, VECTOR_DISTANCE(v, 
          (SELECT v FROM genbvec WHERE name='C1'), HAMMING) DISTANCE
        FROM genbvec
        ORDER BY VECTOR_DISTANCE(v, (SELECT v FROM genbvec WHERE name='C1'), HAMMING);
        
        ```

Example output: 
        
                ```
        NAME     V                  BV                                  DISTANCE
        ________ __________________ ___________________________________ ___________
        C1       [116,26,26,89]     00101110010110000101100010011010    0.0
        C1-6     [116,90,26,89]     00101110010110100101100010011010    1.0
        C1-16    [244,26,26,89]     00101111010110000101100010011010    1.0
        C1-15    [244,26,26,89]     00101111010110000101100010011010    1.0
        C1-12    [124,26,26,89]     00111110010110000101100010011010    1.0
        C1-10    [116,26,24,89]     00101110010110000001100010011010    1.0
        C1-8     [116,26,26,25]     00101110010110000101100010011000    1.0
        C1-1     [116,90,26,88]     00101110010110100101100000011010    2.0
        C1-14    [116,155,26,89]    00101110110110010101100010011010    2.0
        C1-13    [116,24,58,89]     00101110000110000101110010011010    2.0
        C1-11    [100,10,26,89]     00100110010100000101100010011010    2.0
        C1-7     [116,18,26,217]    00101110010010000101100010011011    2.0
        C1-5     [100,30,26,89]     00100110011110000101100010011010    2.0
        C1-4     [116,10,26,73]     00101110010100000101100010010010    2.0
        
        NAME     V                   BV                                  DISTANCE
        ________ ___________________ ___________________________________ ___________
        C1-2     [118,90,26,89]      01101110010110100101100010011010    2.0
        C1-3     [117,91,26,89]      10101110110110100101100010011010    3.0
        C1-9     [124,22,26,89]      00111110011010000101100010011010    3.0
        C1-17    [116,154,58,91]     00101110010110010101110011011010    3.0
        C1-18    [54,30,26,89]       01101100011110000101100010011010    3.0
        C1-19    [52,26,90,25]       00101100010110000101101010011000    3.0
        C1-20    [116,26,18,92]      00101110010110000100100000111010    3.0
        C2-6     [244,77,206,114]    00101111101100100111001101001110    14.0
        C2-19    [244,77,78,50]      00101111101100100111001001001100    14.0
        C2-5     [180,93,206,179]    00101101101110100111001111001101    15.0
        C2-12    [180,13,206,50]     00101101101100000111001101001100    15.0
        C2-11    [252,73,206,50]     00111111100100100111001101001100    15.0
        C2-7     [180,77,206,114]    00101101101100100111001101001110    15.0
        C2       [180,77,206,50]     00101101101100100111001101001100    16.0
        
        NAME     V                   BV                                  DISTANCE
        ________ ___________________ ___________________________________ ___________
        C2-14    [180,221,206,50]    00101101101110110111001101001100    16.0
        C2-4     [148,77,206,114]    00101001101100100111001101001110    16.0
        C2-1     [180,77,202,34]     00101101101100100101001101000100    16.0
        C2-2     [181,93,206,178]    10101101101110100111001101001101    17.0
        C2-18    [182,77,206,50]     01101101101100100111001101001100    17.0
        C2-17    [180,77,204,50]     00101101101100100011001101001100    17.0
        C2-13    [52,77,207,178]     00101100101100101111001101001101    17.0
        C2-10    [180,77,206,54]     00101101101100100111001101101100    17.0
        C2-3     [148,77,198,50]     00101001101100100110001101001100    18.0
        C2-20    [188,205,206,50]    00111101101100110111001101001100    18.0
        C2-15    [182,77,206,54]     01101101101100100111001101101100    18.0
        C2-8     [182,77,238,50]     01101101101100100111011101001100    18.0
        C2-9     [176,77,198,54]     00001101101100100110001101101100    19.0
        C2-16    [181,77,238,34]     10101101101100100111011101000100    19.0
        
        
        42 rows selected.
        ```

    4. Run another similarity search, this time limiting the results to the first 21 rows. In this example, this means the results include only binary vectors from cluster 1\. 
        
                ```
        SELECT name, v, bv, VECTOR_DISTANCE(v, 
            (SELECT v FROM genbvec WHERE name='C1'), HAMMING) DISTANCE
        FROM genbvec
        ORDER BY VECTOR_DISTANCE(v, (SELECT v FROM genbvec WHERE name='C1'), HAMMING)
        FETCH EXACT FIRST 21 ROWS ONLY;
        ```

Example output: 
        
                ```
        NAME     V                  BV                                  DISTANCE
        ________ __________________ ___________________________________ ___________
        C1       [116,26,26,89]     00101110010110000101100010011010    0.0
        C1-6     [116,90,26,89]     00101110010110100101100010011010    1.0
        C1-16    [244,26,26,89]     00101111010110000101100010011010    1.0
        C1-15    [244,26,26,89]     00101111010110000101100010011010    1.0
        C1-12    [124,26,26,89]     00111110010110000101100010011010    1.0
        C1-10    [116,26,24,89]     00101110010110000001100010011010    1.0
        C1-8     [116,26,26,25]     00101110010110000101100010011000    1.0
        C1-1     [116,90,26,88]     00101110010110100101100000011010    2.0
        C1-14    [116,155,26,89]    00101110110110010101100010011010    2.0
        C1-13    [116,24,58,89]     00101110000110000101110010011010    2.0
        C1-11    [100,10,26,89]     00100110010100000101100010011010    2.0
        C1-7     [116,18,26,217]    00101110010010000101100010011011    2.0
        C1-5     [100,30,26,89]     00100110011110000101100010011010    2.0
        C1-4     [116,10,26,73]     00101110010100000101100010010010    2.0
        
        NAME     V                  BV                                  DISTANCE
        ________ __________________ ___________________________________ ___________
        C1-2     [118,90,26,89]     01101110010110100101100010011010    2.0
        C1-3     [117,91,26,89]     10101110110110100101100010011010    3.0
        C1-9     [124,22,26,89]     00111110011010000101100010011010    3.0
        C1-17    [116,154,58,91]    00101110010110010101110011011010    3.0
        C1-18    [54,30,26,89]      01101100011110000101100010011010    3.0
        C1-19    [52,26,90,25]      00101100010110000101101010011000    3.0
        C1-20    [116,26,18,92]     00101110010110000100100000111010    3.0
        
        21 rows selected.
        ```

    5. In this iteration, the similarity search omits the ` HAMMING ` distance metric. However, because ` HAMMING ` is the default metric used with ` BINARY ` vectors, the results are the same as the previous query. 
        
                ```
        SELECT name, v, bv, VECTOR_DISTANCE(v, 
            (SELECT v FROM genbvec WHERE name='C1')) DISTANCE
        FROM genbvec
        ORDER BY VECTOR_DISTANCE(v, (SELECT v FROM genbvec WHERE name='C1'))
        FETCH EXACT FIRST 21 ROWS ONLY;
        
        ```

Example output: 
        
                ```
        NAME     V                  BV                                  DISTANCE
        ________ __________________ ___________________________________ ___________
        C1       [116,26,26,89]     00101110010110000101100010011010    0.0
        C1-6     [116,90,26,89]     00101110010110100101100010011010    1.0
        C1-16    [244,26,26,89]     00101111010110000101100010011010    1.0
        C1-15    [244,26,26,89]     00101111010110000101100010011010    1.0
        C1-12    [124,26,26,89]     00111110010110000101100010011010    1.0
        C1-10    [116,26,24,89]     00101110010110000001100010011010    1.0
        C1-8     [116,26,26,25]     00101110010110000101100010011000    1.0
        C1-1     [116,90,26,88]     00101110010110100101100000011010    2.0
        C1-14    [116,155,26,89]    00101110110110010101100010011010    2.0
        C1-13    [116,24,58,89]     00101110000110000101110010011010    2.0
        C1-11    [100,10,26,89]     00100110010100000101100010011010    2.0
        C1-7     [116,18,26,217]    00101110010010000101100010011011    2.0
        C1-5     [100,30,26,89]     00100110011110000101100010011010    2.0
        C1-4     [116,10,26,73]     00101110010100000101100010010010    2.0
        
        NAME     V                  BV                                  DISTANCE
        ________ __________________ ___________________________________ ___________
        C1-2     [118,90,26,89]     01101110010110100101100010011010    2.0
        C1-3     [117,91,26,89]     10101110110110100101100010011010    3.0
        C1-9     [124,22,26,89]     00111110011010000101100010011010    3.0
        C1-17    [116,154,58,91]    00101110010110010101110011011010    3.0
        C1-18    [54,30,26,89]      01101100011110000101100010011010    3.0
        C1-19    [52,26,90,25]      00101100010110000101101010011000    3.0
        C1-20    [116,26,18,92]     00101110010110000100100000111010    3.0
        
        21 rows selected.
        ```

**Parent topic:** [ Get Started ](get-started-node.html)