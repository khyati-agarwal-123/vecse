## Load Binary Vector Data Using SQL*Loader Example {#GUID-FA53AD95-DFFE-475B-93E5-CF59853A9317}

In this example, you can see how to use SQL*Loader to load binary vector data files.

The vectors in a binary (`fvec`) file are stored in raw 32-bit Little Endian format. 

Each vector takes 4+*`d`**4 bytes for the `.fvecs` file where the first 4 bytes indicate the dimensionality (*`d`*) of the vector (that is, the number of dimensions in the vector) followed by *`d`**4 bytes representing the vector data, as described in the following table: 

**Table: Fields for Vector Dimensions and Components**

Field | Field Type | Description  
---|---|--- *`d`*  | int  | The vector dimension  
components |  array of floats |  The vector components  
  
For binary `fvec` files, they must be defined as follows: 

  * You must specify `LOBFILE`. 
  * You must specify the syntax format `fvecs` to indicate that the dafafile contains binary dimensions. 
  * You must specify that the datafile contains raw binary data (`raw`). 



The following is an example of a control file used to load `VECTOR` columns from binary floating point arrays using the galaxies vector example described in [Understand Hierarchical Navigable Small World Indexes](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-23E74FFC-B6A5-4374-9813-F2634AC43410), but in this case importing `fvecs` data, using the control file syntax `format "fvecs"`: 

> **note:** 

SQL*Loader supports loading `VECTOR` columns from character data and binary floating point array `fvec` files. The format for `fvec` files is that each binary 32-bit floating point array is preceded by a four (4) byte value, which is the number of elements in the vector. There can be multiple vectors in the file, possibly with different dimensions. 
```
    LOAD DATA
    infile 'galaxies_vec.csv'
    INTO TABLE galaxies
    fields terminated by ':'
    trailing nullcols
    (
    id,
    name char (50),
    doc  char (500),
    embedding lobfile (constant '/u01/data/vector/embedding.fvecs' format "fvecs")  raw
    )
```
    

The data contained in `galaxies_vec.csv` in this case does not have the vector data. Instead, the vector data will be read from the secondary `LOBFILE` in the `/u01/data/vector` directory (`/u01/data/vector/embedding.fvecs`), which contains the same information in `float32` floating point binary numbers, but is in `fvecs` format: 
```
    1:M31:Messier 31 is a barred spiral galaxy in the Andromeda constellation which has a lot of barred spiral galaxies.:
    2:M33:Messier 33 is a spiral galaxy in the Triangulum constellation.:
    3:M58:Messier 58 is an intermediate barred spiral galaxy in the Virgo constellation.:
    4:M63:Messier 63 is a spiral galaxy in the Canes Venatici constellation.:
    5:M77:Messier 77 is a barred spiral galaxy in the Cetus constellation.:
    6:M91:Messier 91 is a barred spiral galaxy in the Coma Berenices constellation.:
    7:M49:Messier 49 is a giant elliptical galaxy in the Virgo constellation.:
    8:M60:Messier 60 is an elliptical galaxy in the Virgo constellation.:
    9:NGC1073:NGC 1073 is a barred spiral galaxy in Cetus constellation.:
```
    

**Parent topic:** [Load Vector Data Using SQL*Loader](load-vector-data-using-sqlloader.md)
