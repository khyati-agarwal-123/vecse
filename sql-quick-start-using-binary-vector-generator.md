## SQL Quick Start Using a BINARY Vector Generator {#GUID-28958BF9-9294-406B-B47F-C800D38F4C07}

A set of procedures generate `BINARY` vectors, providing a simple way to get started with Oracle AI Vector Search without a vector embedding model. 

The included procedures allow you to randomly generate binary vectors with a specified number of dimensions and clusters. The output of the generation process is the population of a table called `genbvec` that you can then use, for example, to experiment with similarity searches. 

The following instructions assume you already have access to a database account with sufficient privileges (minimally the `DB_DEVELOPER_ROLE` role). 

> **note:** Do not use the `BINARY` vector generator on production databases. This tutorial is made available for testing and demo purposes. 

> **note:** If you already have access to a third-party `BINARY` vector embedding model, you can perform a real text-to-`BINARY`-embedding transformation by calling third-party REST APIs using the Vector Utility PL/SQL package `DBMS_VECTOR`. For more information, refer to the example in [Convert Text String to BINARY Embedding Outside Oracle Database](convert-text-string-binary-embedding-oracle-database.md#GUID-425922B4-C9DA-48B6-B98F-7846D53AE0AF). 

  1. Create the `genbvec` table.
```
    DROP TABLE genbvec PURGE;
    
    CREATE TABLE genbvec (
    id NUMBER,            -- id of the generated vector
    v VECTOR(*, BINARY),  -- generated vector
    name VARCHAR2(500),   -- name for the generated vector: C1 to Cn are centroids, Cx_y is vector number y in cluster number x
    bv VARCHAR2(40),      -- bit version of the generated vector
    ly NUMBER             -- random number you can use to filter out rows in addiion to similarity search on vectors
    );
```
    

  2. Create the procedures used to generate `BINARY` vectors:
```
    CREATE OR REPLACE PROCEDURE **generate_random_binary_vector**(
    dimensions IN NUMBER,
    result_int OUT VECTOR,
    result_binary OUT VARCHAR2
    ) IS
    binary_vector VARCHAR2(40);
    int8_value NUMBER;
    number_of_bits NUMBER;
    char_vector VARCHAR2(40);
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
    result_int := to_vector(char_vector, dimensions, BINARY);
    result_binary := binary_vector;
    END generate_random_binary_vector;
    /
```
```
    CREATE OR REPLACE PROCEDURE **generate_binary_cluster**(
    centroid IN VARCHAR2,            -- a string of 1 and 0
    spread IN NUMBER,                -- Maximum Hamming distance between centroid and other vectors in the same cluster
    cluster_size IN NUMBER,          -- Number of vectors to generate in addition to the centroid
    result_binary OUT SYS_REFCURSOR,
    result_int8 OUT SYS_REFCURSOR
    ) IS
    dimension NUMBER;
    max_spread NUMBER;
    vector VARCHAR2(40);
    char_vector VARCHAR2(40);
    flip_positions DBMS_SQL.VARCHAR2_TABLE;
    random_position NUMBER;
    tresult_binary DBMS_SQL.VARCHAR2_TABLE;
    tresult_int8 DBMS_SQL.VARCHAR2_TABLE;
    binary_vector VARCHAR2(40);
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
    CREATE OR REPLACE PROCEDURE **generate_binary_vectors**(
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
    ri VECTOR(*, BINARY);
    rb VARCHAR2(40);
    result_binary SYS_REFCURSOR;
    result_int8 SYS_REFCURSOR;
    vb VARCHAR2(40);
    vi VARCHAR2(40);
    BEGIN
    
    IF (num_vectors) <=0 OR (num_clusters < 1) OR (num_vectors < num_clusters)
    OR (dimensions <= 0) OR (dimensions > 504) OR (cluster_spread <= 0) THEN
    RAISE_APPLICATION_ERROR(-20001, 'Issues with arguments provided');
    END IF;
    
    SELECT MAX(id) INTO max_id FROM genbvec;
    IF max_id IS NULL THEN max_id := 0;
    END IF;
    
    -- Calculate vectors per cluster
    vectors_per_cluster := TRUNC(num_vectors / num_clusters);
    remaining_vectors := num_vectors MOD num_clusters; -- remaining vectors are not generated
    
    -- Generate cluster centroids
    FOR i IN 1..num_clusters LOOP
    generate_random_binary_vector(dimensions, ri, rb);
    INSERT INTO genbvec VALUES (max_id + idx, ri, 'C'||i, rb, DBMS_RANDOM.VALUE(3,600000000));
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
    ri := TO_VECTOR(vi, dimensions, BINARY);
    INSERT INTO genbvec VALUES (max_id + idx, ri, 'C'||i||'-'||j, vb, DBMS_RANDOM.VALUE(3,600000000));
    j := j+1;
    idx := idx + 1;
    END LOOP;
    CLOSE result_binary;
    CLOSE result_int8;
    END IF;
    END LOOP;
    COMMIT;
    
    END generate_binary_vectors;
    /
```
    

  3. After you have your vector generator procedures set up, you can run the commands in this step to get started experimenting with `BINARY` vectors in the database.

This example generates two clusters, each having twenty-one 32-dimension vectors (including the centroid) with a maximum spread of 3 from the centroid:

    1. Start out by generating some `BINARY` vectors using the `generate_binary_vectors` procedure. The results of the generation are inserted into the table, `genbvec`.
```
        BEGIN
        generate_binary_vectors(
        num_vectors  =>   40,   -- If numbers of vector is not a multiple of num_clusters, remaining vectors are not generated
        num_clusters =>    2,   -- Must be grather than 0
        dimensions   =>   32,   -- Must be a multiple of 8 and less than 504
        cluster_spread =>  3    -- Maximum Hamming distance between centroid and other vectors in the same cluster: max number of bits flipped
        );
        END;
        /
```
        

    2. Run a `SELECT` statement to view the generated `BINARY` vectors.
```
        SET SERVEROUTPUT ON;
        
        SELECT id, v, name, VECTOR_DIMENSION_COUNT(v) DIMS, VECTOR_DIMENSION_FORMAT(v) FORMAT, bv, ly FROM genbvec;
```
        

Example output:
```
        ID  V                    NAME       DIMS      FORMAT      BV                                               LY
        -------  -------------------  ---------  --------  ----------  -----------------------------------  --------------
        1  [24,153,161,63]      C1         32        BINARY      00011000100110011000010111111100     99789021.1
        2  [24,153,165,63]      C1-1       32        BINARY      00011000100110011010010111111100     60221003.5
        3  [26,152,165,63]      C1-2       32        BINARY      01011000000110011010010111111100     387124796
        4  [24,201,161,62]      C1-3       32        BINARY      00011000100100111000010101111100     291263868
        5  [24,187,161,63]      C1-4       32        BINARY      00011000110111011000010111111100     583827824
        6  [24,153,161,61]      C1-5       32        BINARY      00011000100110011000010110111100     144826451
        7  [24,153,171,55]      C1-6       32        BINARY      00011000100110011101010111101100     113684378
        8  [88,153,161,61]      C1-7       32        BINARY      00011010100110011000010110111100     312081799
        9  [152,217,161,47]     C1-8       32        BINARY      00011001100110111000010111110100     173971628
        10  [24,153,163,59]      C1-9       32        BINARY      00011000100110011100010111011100     500775192
        11  [24,153,160,61]      C1-10      32        BINARY      00011000100110010000010110111100     137309652
        12  [25,185,161,47]      C1-11      32        BINARY      10011000100111011000010111110100     483392712
        13  [89,153,161,63]      C1-12      32        BINARY      10011010100110011000010111111100     458730494
        14  [24,153,229,31]      C1-13      32        BINARY      00011000100110011010011111111000     325738354
        
        ID  V                    NAME       DIMS      FORMAT      BV                                               LY
        -------  -------------------  ---------  --------  ----------  -----------------------------------  --------------
        15  [24,152,161,63]      C1-14      32        BINARY      00011000000110011000010111111100     260267242
        16  [24,153,165,63]      C1-15      32        BINARY      00011000100110011010010111111100     153663322
        17  [24,137,169,63]      C1-16      32        BINARY      00011000100100011001010111111100     411918780
        18  [24,185,161,63]      C1-17      32        BINARY      00011000100111011000010111111100     53885587.1
        19  [152,137,161,63]     C1-18      32        BINARY      00011001100100011000010111111100     321305137
        20  [25,153,161,63]      C1-19      32        BINARY      10011000100110011000010111111100     180742593
        21  [16,153,161,63]      C1-20      32        BINARY      00001000100110011000010111111100     511768659
        22  [183,107,24,190]     C2         32        BINARY      11101101110101100001100001111101     529205377
        23  [181,251,24,190]     C2-1       32        BINARY      10101101110111110001100001111101     391560729
        24  [191,107,25,186]     C2-2       32        BINARY      11111101110101101001100001011101     191852938
        25  [182,106,24,190]     C2-3       32        BINARY      01101101010101100001100001111101     164088550
        26  [183,107,56,187]     C2-4       32        BINARY      11101101110101100001110011011101     20400437.6
        27  [183,106,16,190]     C2-5       32        BINARY      11101101010101100000100001111101     363725396
        28  [183,107,40,190]     C2-6       32        BINARY      11101101110101100001010001111101     144549103
        
        ID  V                    NAME       DIMS      FORMAT      BV                                               LY
        -------  -------------------  ---------  --------  ----------  -----------------------------------  --------------
        29  [183,107,26,190]     C2-7       32        BINARY      11101101110101100101100001111101     318036129
        30  [183,123,24,188]     C2-8       32        BINARY      11101101110111100001100000111101     309460286
        31  [179,35,24,190]      C2-9       32        BINARY      11001101110001000001100001111101     25042254.7
        32  [182,251,24,190]     C2-10      32        BINARY      01101101110111110001100001111101     355499793
        33  [183,251,24,190]     C2-11      32        BINARY      11101101110111110001100001111101     483002129
        34  [183,107,24,254]     C2-12      32        BINARY      11101101110101100001100001111111     497697267
        35  [183,42,24,158]      C2-13      32        BINARY      11101101010101000001100001111001     64446273.5
        36  [151,107,28,186]     C2-14      32        BINARY      11101001110101100011100001011101     248483969
        37  [167,43,16,190]      C2-15      32        BINARY      11100101110101000000100001111101     513880134
        38  [183,106,24,190]     C2-16      32        BINARY      11101101010101100001100001111101     558247180
        39  [183,123,24,190]     C2-17      32        BINARY      11101101110111100001100001111101     287706546
        40  [151,107,24,190]     C2-18      32        BINARY      11101001110101100001100001111101     309138884
        41  [167,107,28,186]     C2-19      32        BINARY      11100101110101100011100001011101     433932877
        42  [63,106,24,190]      C2-20      32        BINARY      11111100010101100001100001111101     84539416.7
```
        

    3. Perform a similarity search on the `BINARY` vectors.
```
        SELECT name, v, bv, VECTOR_DISTANCE(v,
        (SELECT v FROM genbvec WHERE name='C1'), HAMMING) DISTANCE
        FROM genbvec
        ORDER BY VECTOR_DISTANCE(v, (SELECT v FROM genbvec WHERE name='C1'), HAMMING);
```
        

Example output:
```
        NAME  V                  BV                                 DISTANCE
        -------  -----------------  ---------------------------------  ----------
        C1  [24,153,161,63]    00011000100110011000010111111100   0
        C1-1  [24,153,165,63]    00011000100110011010010111111100   1.0
        C1-20  [16,153,161,63]    00001000100110011000010111111100   1.0
        C1-19  [25,153,161,63]    10011000100110011000010111111100   1.0
        C1-17  [24,185,161,63]    00011000100111011000010111111100   1.0
        C1-15  [24,153,165,63]    00011000100110011010010111111100   1.0
        C1-14  [24,152,161,63]    00011000000110011000010111111100   1.0
        C1-5  [24,153,161,61]    00011000100110011000010110111100   1.0
        C1-4  [24,187,161,63]    00011000110111011000010111111100   2.0
        C1-18  [152,137,161,63]   00011001100100011000010111111100   2.0
        C1-16  [24,137,169,63]    00011000100100011001010111111100   2.0
        C1-12  [89,153,161,63]    10011010100110011000010111111100   2.0
        C1-10  [24,153,160,61]    00011000100110010000010110111100   2.0
        C1-9  [24,153,163,59]    00011000100110011100010111011100   2.0
        
        NAME  V                  BV                                 DISTANCE
        -------  -----------------  ---------------------------------  ----------
        C1-7  [88,153,161,61]    00011010100110011000010110111100   2.0
        C1-2  [26,152,165,63]    01011000000110011010010111111100   3.0
        C1-3  [24,201,161,62]    00011000100100111000010101111100   3.0
        C1-6  [24,153,171,55]    00011000100110011101010111101100   3.0
        C1-8  [152,217,161,47]   00011001100110111000010111110100   3.0
        C1-11  [25,185,161,47]    10011000100111011000010111110100   3.0
        C1-13  [24,153,229,31]    00011000100110011010011111111000   3.0
        C2-1  [181,251,24,190]   10101101110111110001100001111101   15.0
        C2-10  [182,251,24,190]   01101101110111110001100001111101   15.0
        C2-6  [183,107,40,190]   11101101110101100001010001111101   16.0
        C2-11  [183,251,24,190]   11101101110111110001100001111101   16.0
        C2-2  [191,107,25,186]   11111101110101101001100001011101   17.0
        C2-20  [63,106,24,190]    11111100010101100001100001111101   17.0
        C2-18  [151,107,24,190]   11101001110101100001100001111101   17.0
        
        NAME  V                  BV                                 DISTANCE
        -------  -----------------  ---------------------------------  ----------
        C2-17  [183,123,24,190]   11101101110111100001100001111101   17.0
        C2-15  [167,43,16,190]    11100101110101000000100001111101   17.0
        C2-9  [179,35,24,190]    11001101110001000001100001111101   17.0
        C2-4  [183,107,56,187]   11101101110101100001110011011101   17.0
        C2  [183,107,24,190]   11101101110101100001100001111101   18.0
        C2-8  [183,123,24,188]   11101101110111100001100000111101   18.0
        C2-3  [182,106,24,190]   01101101010101100001100001111101   18.0
        C2-5  [183,106,16,190]   11101101010101100000100001111101   18.0
        C2-7  [183,107,26,190]   11101101110101100101100001111101   19.0
        C2-12  [183,107,24,254]   11101101110101100001100001111111   19.0
        C2-13  [183,42,24,158]    11101101010101000001100001111001   19.0
        C2-14  [151,107,28,186]   11101001110101100011100001011101   19.0
        C2-16  [183,106,24,190]   11101101010101100001100001111101   19.0
        C2-19  [167,107,28,186]   11100101110101100011100001011101   21.0
        
        42 rows selected.
```
        

    4. Run another similarity search, this time limiting the results to the first 21 rows. In this example, this means the results include `BINARY` vectors only from cluster 1.
```
        SELECT name, v, bv, VECTOR_DISTANCE(v,
        (SELECT v FROM genbvec WHERE name='C1'), HAMMING) DISTANCE
        FROM genbvec
        ORDER BY VECTOR_DISTANCE(v, (SELECT v FROM genbvec WHERE name='C1'), HAMMING)
        FETCH EXACT FIRST 21 ROWS ONLY;
```
        

Example output:
```
        NAME  V                   BV                                 DISTANCE
        -------  ------------------  ---------------------------------  ----------
        C1  [24,153,161,63]     00011000100110011000010111111100   0
        C1-1  [24,153,165,63]     00011000100110011010010111111100   1.0
        C1-20  [16,153,161,63]     00001000100110011000010111111100   1.0
        C1-19  [25,153,161,63]     10011000100110011000010111111100   1.0
        C1-17  [24,185,161,63]     00011000100111011000010111111100   1.0
        C1-15  [24,153,165,63]     00011000100110011010010111111100   1.0
        C1-14  [24,152,161,63]     00011000000110011000010111111100   1.0
        C1-5  [24,153,161,61]     00011000100110011000010110111100   1.0
        C1-4  [24,187,161,63]     00011000110111011000010111111100   2.0
        C1-18  [152,137,161,63]    00011001100100011000010111111100   2.0
        C1-16  [24,137,169,63]     00011000100100011001010111111100   2.0
        C1-12  [89,153,161,63]     10011010100110011000010111111100   2.0
        C1-10  [24,153,160,61]     00011000100110010000010110111100   2.0
        C1-9  [24,153,163,59]     00011000100110011100010111011100   2.0
        
        NAME  V                   BV                                 DISTANCE
        -------  ------------------  ---------------------------------  ----------
        C1-7  [88,153,161,61]     00011010100110011000010110111100   2.0
        C1-2  [26,152,165,63]     01011000000110011010010111111100   3.0
        C1-3  [24,201,161,62]     00011000100100111000010101111100   3.0
        C1-6  [24,153,171,55]     00011000100110011101010111101100   3.0
        C1-8  [152,217,161,47]    00011001100110111000010111110100   3.0
        C1-11  [25,185,161,47]     10011000100111011000010111110100   3.0
        C1-13  [24,153,229,31]     00011000100110011010011111111000   3.0
        
        21 rows selected.
```
        

    5. In this iteration, the similarity search omits the `HAMMING` distance metric. However, because `HAMMING` is the default metric used with `BINARY` vectors, the results are the same as the previous query.
```
        SELECT name, v, bv, VECTOR_DISTANCE(v,
        (SELECT v FROM genbvec WHERE name='C1')) DISTANCE
        FROM genbvec
        ORDER BY VECTOR_DISTANCE(v, (SELECT v FROM genbvec WHERE name='C1'))
        FETCH EXACT FIRST 21 ROWS ONLY;
```
        

Example output:
```
        NAME  V                   BV                                 DISTANCE
        -------  ------------------  ---------------------------------  ----------
        C1  [24,153,161,63]     00011000100110011000010111111100   0
        C1-1  [24,153,165,63]     00011000100110011010010111111100   1.0
        C1-20  [16,153,161,63]     00001000100110011000010111111100   1.0
        C1-19  [25,153,161,63]     10011000100110011000010111111100   1.0
        C1-17  [24,185,161,63]     00011000100111011000010111111100   1.0
        C1-15  [24,153,165,63]     00011000100110011010010111111100   1.0
        C1-14  [24,152,161,63]     00011000000110011000010111111100   1.0
        C1-5  [24,153,161,61]     00011000100110011000010110111100   1.0
        C1-4  [24,187,161,63]     00011000110111011000010111111100   2.0
        C1-18  [152,137,161,63]    00011001100100011000010111111100   2.0
        C1-16  [24,137,169,63]     00011000100100011001010111111100   2.0
        C1-12  [89,153,161,63]     10011010100110011000010111111100   2.0
        C1-10  [24,153,160,61]     00011000100110010000010110111100   2.0
        C1-9  [24,153,163,59]     00011000100110011100010111011100   2.0
        
        NAME  V                   BV                                 DISTANCE
        -------  ------------------  ---------------------------------  ----------
        C1-7  [88,153,161,61]     00011010100110011000010110111100   2.0
        C1-2  [26,152,165,63]     01011000000110011010010111111100   3.0
        C1-3  [24,201,161,62]     00011000100100111000010101111100   3.0
        C1-6  [24,153,171,55]     00011000100110011101010111101100   3.0
        C1-8  [152,217,161,47]    00011001100110111000010111110100   3.0
        C1-11  [25,185,161,47]     10011000100111011000010111110100   3.0
        C1-13  [24,153,229,31]     00011000100110011010011111111000   3.0
        
        21 rows selected.
```
        




**Parent topic:** [Get Started](get-started-node.md)
