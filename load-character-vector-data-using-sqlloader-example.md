## Load Character Vector Data Using SQL*Loader Example {#GUID-F9994F41-B4D7-421E-A846-366ED74E3CDF}

In this example, you can see how to use SQL*Loader to load vector data into a five-dimension vector space.

Let's imagine we have the following text documents classifying galaxies by their types:

  * **DOC1**: "Messier 31 is a barred spiral galaxy in the Andromeda constellation which has a lot of barred spiral galaxies." 
  * **DOC2**: "Messier 33 is a spiral galaxy in the Triangulum constellation." 
  * **DOC3**: "Messier 58 is an intermediate barred spiral galaxy in the Virgo constellation." 
  * **DOC4**: "Messier 63 is a spiral galaxy in the Canes Venatici constellation." 
  * **DOC5**: "Messier 77 is a barred spiral galaxy in the Cetus constellation." 
  * **DOC6**: "Messier 91 is a barred spiral galaxy in the Coma Berenices constellation." 
  * **DOC7**: "NGC 1073 is a barred spiral galaxy in Cetus constellation." 
  * **DOC8**: "Messier 49 is a giant elliptical galaxy in the Virgo constellation." 
  * **DOC9**: "Messier 60 is an elliptical galaxy in the Virgo constellation." 



You can create vectors representing the preceding galaxy's classes using the following five-dimension vector space based on the count of important words appearing in each document:

**Table: Five dimension vector space**

Galaxy Classes | Intermediate | Barred | Spiral | Giant | Elliptical  
---|---|---|---|---|---  
M31 | 0 | 2 | 2 | 0 | 0  
M33 | 0 | 0 | 1 | 0 | 0  
M58 | 1 | 1 | 1 | 0 | 0  
M63 | 0 | 0 | 1 | 0 | 0  
M77 | 0 | 1 | 1 | 0 | 0  
M91 | 0 | 1 | 1 | 0 | 0  
M49 | 0 | 0 | 0 | 1 | 1  
M60 | 0 | 0 | 0 | 0 | 1  
NGC1073 | 0 | 1 | 1 | 0 | 0  
  
This naturally gives you the following vectors:

  * **M31**: `[0,2,2,0,0]`
  * **M33**: `[0,0,1,0,0]`
  * **M58**: `[1,1,1,0,0]`
  * **M63**: `[0,0,1,0,0]`
  * **M77**: `[0,1,1,0,0]`
  * **M91**: `[0,1,1,0,0]`
  * **M49**: `[0,0,0,1,1]`
  * **M60**: `[0,0,0,0,1]`
  * **NGC1073**: `[0,1,1,0,0]`



You can use SQL*Loader to load this data into the `GALAXIES` database table defined as: 
```
    drop table galaxies purge;
    create table galaxies (id number, name varchar2(50), doc varchar2(500), embedding vector);
```
    

Based on the data described previously, you can create the following `galaxies_vec.csv` file: 
```
    1:M31:Messier 31 is a barred spiral galaxy in the Andromeda constellation which has a lot of barred spiral galaxies.:[0,2,2,0,0]:
    2:M33:Messier 33 is a spiral galaxy in the Triangulum constellation.:[0,0,1,0,0]:
    3:M58:Messier 58 is an intermediate barred spiral galaxy in the Virgo constellation.:[1,1,1,0,0]:
    4:M63:Messier 63 is a spiral galaxy in the Canes Venatici constellation.:[0,0,1,0,0]:
    5:M77:Messier 77 is a barred spiral galaxy in the Cetus constellation.:[0,1,1,0,0]:
    6:M91:Messier 91 is a barred spiral galaxy in the Coma Berenices constellation.:[0,1,1,0,0]:
    7:M49:Messier 49 is a giant elliptical galaxy in the Virgo constellation.:[0,0,0,1,1]:
    8:M60:Messier 60 is an elliptical galaxy in the Virgo constellation.:[0,0,0,0,1]:
    9:NGC1073:NGC 1073 is a barred spiral galaxy in Cetus constellation.:[0,1,1,0,0]:
```
    

Here is a possible SQL*Loader control file `galaxies_vec.ctl`: 
```
    recoverable
    LOAD DATA
    infile 'galaxies_vec.csv'
    INTO TABLE galaxies
    fields terminated by ':'
    trailing nullcols
    (
    id,
    name char (50),
    doc  char (500),
    embedding char (32000)
    )
```
    

> **note:** You cannot use comma-delimited vectors (vectors separated by commas) as the field terminator in your CSV file. You must use another deliminator. In these examples the deliminator is a colon (`:`). 

After you have created the two files `galaxies_vec.csv` and `galaxies_vec.ctl`, you can run the following sequence of instructions directly from your favorite SQL command line tool: 
```
    host sqlldr vector/vector@CDB1_PDB1 control=galaxies_vec.ctl log=galaxies_vec.log
    
    SQL*Loader: Release 23.0.0.0.0 - Development on Thu Jan 11 19:46:21 2024
    Version 23.4.0.23.00
    
    Copyright (c) 1982, 2024, Oracle and/or its affiliates.  All rights reserved.
    
    Path used:      Conventional
    Commit point reached - logical record count 10
    
    Table GALAXIES2:
    9 Rows successfully loaded.
    
    Check the log file:
    galaxies_vec.log
    for more information about the load.
    
    SQL>
    
    select * from galaxies;
    
    ID NAME    DOC                                                        EMBEDDING
    --- ------  ---------------------------------------------------------- ---------------------------------
    1 M31     Messier 31 is a barred spiral galaxy in the Andromeda ... [0,2.0E+000,2.0E+000,0,0]
    2 M33     Messier 33 is a spiral galaxy in the Triangulum ...       [0,0,1.0E+000,0,0]
    3 M58     Messier 58 is an intermediate barred spiral galaxy ...    [1.0E+000,1.0E+000,1.0E+000,0,0]
    4 M63     Messier 63 is a spiral galaxy in the Canes Venatici ...   [0,0,1.0E+000,0,0]
    5 M77     Messier 77 is a barred spiral galaxy in the Cetus ...     [0,1.0E+000,1.0E+000,0,0]
    6 M91     Messier 91 is a barred spiral galaxy in the Coma ...      [0,1.0E+000,1.0E+000,0,0]
    7 M49     Messier 49 is a giant elliptical galaxy in the Virgo ...  [0,0,0,1.0E+000,1.0E+000]
    8 M60     Messier 60 is an elliptical galaxy in the Virgo ...       [0,0,0,0,1.0E+000]
    9 NGC1073 NGC 1073 is a barred spiral galaxy in Cetus ...           [0,1.0E+000,1.0E+000,0,0]
    
    9 rows selected.
    
    SQL>
```
    

Here is the resulting log file for this load (`galaxies_vec.log`): 
```
    cat galaxies_vec.log
    
    SQL*Loader: Release 23.0.0.0.0 - Development on Thu Jan 11 19:46:21 2024
    Version 23.4.0.23.00
    
    Copyright (c) 1982, 2024, Oracle and/or its affiliates.  All rights reserved.
    
    Control File:   galaxies_vec.ctl
    Data File:      galaxies_vec.csv
    Bad File:     galaxies_vec.bad
    Discard File:  none specified
    
    (Allow all discards)
    
    Number to load: ALL
    Number to skip: 0
    Errors allowed: 50
    Bind array:     250 rows, maximum of 1048576 bytes
    Continuation:    none specified
    Path used:      Conventional
    
    Table GALAXIES, loaded from every logical record.
    Insert option in effect for this table: INSERT
    TRAILING NULLCOLS option in effect
    
    Column Name   Position   Len  Term Encl Datatype
    ----------- ---------- ----- ---- ---- ----------
    ID               FIRST     *   :       CHARACTER
    NAME              NEXT    50   :       CHARACTER
    DOC               NEXT   500   :       CHARACTER
    EMBEDDING         NEXT 32000   :       CHARACTER
    
    value used for ROWS parameter changed from 250 to 31
    
    
    Table GALAXIES2:
    9 Rows successfully loaded.
    0 Rows not loaded due to data errors.
    0 Rows not loaded because all WHEN clauses were failed.
    
    
    Space allocated for bind array:                1017234 bytes(31 rows)
    Read   buffer bytes: 1048576
    
    Total logical records skipped:          0
    Total logical records read:             9
    Total logical records rejected:         0
    Total logical records discarded:        1
    
    Run began on Thu Jan 11 19:46:21 2024
    Run ended on Thu Jan 11 19:46:24 2024
    
    Elapsed time was:     00:00:02.43
    CPU time was:         00:00:00.03
    $
```
    

> **note:** This example uses `embedding char (32000)` vectors. For very large vectors, you can use the `LOBFILE` feature 

**Related Topics**

  * [Loading LOB Data from LOBFILEs](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=SUTIL-GUID-E02C2828-ABD1-4B8D-9561-124D221B4BE3)



**Parent topic:** [Load Vector Data Using SQL*Loader](load-vector-data-using-sqlloader.md)
