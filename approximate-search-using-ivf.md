## Approximate Search Using IVF {#GUID-AB45B8A4-26D2-49CC-B807-C1B009B6644B}

This example shows how you can create the Inverted File Flat (IVF) index and run an approximate search using that index.
```
    create table galaxies (id number, name varchar2(50), doc varchar2(500), embedding vector(5,INT8));
    
    insert into galaxies values (1, 'M31', 'Messier 31 is a barred spiral galaxy in the Andromeda constellation which has a lot of barred spiral galaxies.', '[0,2,2,0,0]');
    insert into galaxies values (2, 'M33', 'Messier 33 is a spiral galaxy in the Triangulum constellation.', '[0,0,1,0,0]');
    insert into galaxies values (3, 'M58', 'Messier 58 is an intermediate barred spiral galaxy in the Virgo constellation.', '[1,1,1,0,0]');
    insert into galaxies values (4, 'M63', 'Messier 63 is a spiral galaxy in the Canes Venatici constellation.', '[0,0,1,0,0]');
    insert into galaxies values (5, 'M77', 'Messier 77 is a barred spiral galaxy in the Cetus constellation.', '[0,1,1,0,0]');
    insert into galaxies values (6, 'M91', 'Messier 91 is a barred spiral galaxy in the Coma Berenices constellation.', '[0,1,1,0,0]');
    insert into galaxies values (7, 'M49', 'Messier 49 is a giant elliptical galaxy in the Virgo constellation.', '[0,0,0,1,1]');
    insert into galaxies values (8, 'M60', 'Messier 60 is an elliptical galaxy in the Virgo constellation.', '[0,0,0,0,1]');
    insert into galaxies values (9, 'NGC1073', 'NGC 1073 is a barred spiral galaxy in Cetus constellation.', '[0,1,1,0,0]');
    
    ...
    
    commit;
```
    

In the galaxies example, this is how you can create the IVF index and how you can run an approximate search using that index:
```
    CREATE VECTOR INDEX galaxies_ivf_idx ON galaxies (embedding) ORGANIZATION NEIGHBOR PARTITIONS
    DISTANCE COSINE
    WITH TARGET ACCURACY 95;
    
    SELECT name
    FROM galaxies
    WHERE name <> 'NGC1073'
    ORDER BY VECTOR_DISTANCE( embedding, to_vector('[0,1,1,0,0]'), COSINE )
    FETCH APPROXIMATE FIRST 3 ROWS ONLY;
```
    

If the index is used by the optimizer, then this approximate search example inherits a target accuracy of `95` as it is set when the index is defined. You can override the `TARGET ACCURACY` of your search by running the following query examples: 
```
    SELECT name
    FROM galaxies
    WHERE name <> 'NGC1073'
    ORDER BY VECTOR_DISTANCE( embedding, to_vector('[0,1,1,0,0]'), COSINE )
    FETCH APPROXIMATE FIRST 3 ROWS ONLY WITH TARGET ACCURACY 90;
```
```
    SELECT name
    FROM galaxies
    WHERE name <> 'NGC1073'
    ORDER BY VECTOR_DISTANCE( embedding, to_vector('[0,1,1,0,0]') )
    FETCH APPROXIMATE FIRST 3 ROWS ONLY WITH TARGET ACCURACY PARAMETERS ( NEIGHBOR PARTITION PROBES 10 );
```
    

You can also create the index using the following syntax:
```
    CREATE VECTOR INDEX galaxies_ivf_idx ON galaxies (embedding) ORGANIZATION NEIGHBOR PARTITIONS
    DISTANCE COSINE
    WITH TARGET ACCURACY 90 PARAMETERS (type IVF, neighbor partitions 100);
```
    

> **note:** The `APPROX` and `APPROXIMATE` keywords are optional. If omitted while connected to an ADB-S instance, an approximate search using a vector index is attempted if one exists. 

> **note:** See Also: 

[*Oracle Database SQL Language Reference*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SQLRF-GUID-CFA006CA-6FF1-4972-821E-6996142A51C6) for the full syntax of the `ROW_LIMITING_CLAUSE`

**Parent topic:** [Approximate Similarity Search Examples](approximate-similarity-search-examples.md)
