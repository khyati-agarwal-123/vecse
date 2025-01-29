## BINARY Vectors {#GUID-7DF2FF0C-4E61-47FE-9D5F-953DD006E3F3}

In addition to `FLOAT32` (the default format for comma-separated string representations of vectors), `FLOAT64`, and `INT8` dimension formats, you can also use the `BINARY` dimension format. 

A `BINARY` vector represents each dimension as a single bit (0 or 1). The following statement is an example of declaring a 1024 dimension vector using the `BINARY` format: 
```
    VECTOR(1024, BINARY)
```
    

The main advantages of using the `BINARY` format are: 

  * The storage footprint of vectors can be reduced by a factor of 32 compared to the default `FLOAT32` format. 
  * Distance computations between two vectors are up to 40 times faster.



The downside of using the `BINARY` format is the potential loss of accuracy. However, the loss is often not very substantial. `BINARY` vector accuracy is often greater than 90% compared to that of `FLOAT32` vectors. Several third-party providers have added embedding models that have the ability to generate binary embeddings, including Cohere, Hugging Face, and Jina AI. 

`BINARY` vectors are stored as packed `UINT8` bytes (unsigned integer). This means that a single byte represents exactly 8 `BINARY` dimensions and no less. 

> **note:** 

  * The default distance metric for `BINARY` vectors is `HAMMING`. 

  * Conversions between `BINARY` vectors and any other dimension format are not currently supported. 

  * A column can be declared as `VECTOR(*, BINARY)`. In this case, '`*`' means that vectors can have an arbitrary number of dimensions. However, because the maximum possible number of dimensions supported is 65535 for other formats, you cannot exceed a `UINT8` array of size 8191 for a `BINARY` vector, which represents 8191 * 8 = 65528 dimensions, the greatest multiple of 8 less than 65535. 

  * Oracle does not currently support using OML4Py to export `BINARY` models to ONNX format and import them in the Oracle Database. 

  * Oracle Database server does not currently provide quantization techniques. When applicable, this should be handled by the client.




The following is an example of a Cohere `INT8` embedding and a `UBINARY` embedding. Note when considering this example that Oracle Database server works with the data provided in the `VECTOR` constructor as it is received. Any quantization logic necessary for processing the data into binary format should be handled by the client. The example is provided to help in understanding the concept. 
```
    INT8 Embedding of 1024 dimensions from Cohere embed-english-v3.0:
    [25, 11, -99, -114, 13, -17, -59, 44, 65, 33, -50, -2, 28, -16, -6, -20, -33,
    49, -59, -50, 0, -82, -67, 10, 82, -2, -126, -28, -32, -69, -13, 120, 54, 4,
    -71, 24, 4, -37, -57, 34, 16, -7, 27, -74, -12, 13, 1, -24, 65, -24, 28, 46,
    25, -33, -25, 36, 3, -47, 12, -49, -17, 11, 53, 70, -18, 10, -8, 4, 0, -33,
    10, -3, 27, -24, -35, -24, 23, -32, 0, -4, -21, -7, -29, -48, -7, -28, -25,
    -8, 54, -7, 14, -8, 39, 78, 0, -13, 26, 2, 40, 27, -35, -26, 5, -23, 15, 72,
    -4, -5, 33, 14, 18, 11, 0, -6, 6, -16, -53, 56, -35, 15, -1, -8, 83, 28, -2,
    27, -34, -60, 36, 4, 14, 21, -69, 17, -22, 0, 16, -77, 29, 27, 26, 0, 81, 15,
    -90, 7, 22, -2, -26, -39, -31, -10, 2, 32, -30, 40, -71, 29, 2, 36, -72, -6,
    42, -16, -16, 6, 40, 30, 1, -31, -42, 31, 56, 18, 0, 9, 27, 59, 11, 38, 28,
    -30, 73, -10, -56, 6, 17, 87, 15, 1, 49, -33, -68, 0, 10, -49, 18, -10, 8, 12,
    52, -31, 7, -37, -25, -53, 9, -5, 72, 14, -37, -41, 30, -54, -60, 30, -62, 20,
    3, 7, 64, -7, 48, 16, 19, 1, -43, -18, -91, -6, -113, 104, 42, 61, -24, -15,
    20, -9, 4, 36, 27, 46, -30, -39, 43, -14, 53, -36, -4, 35, 74, 37, 1, -19, 62,
    12, -13, 8, -11, 21, -4, 96, 29, 17, -99, 2, -67, -32, -55, -8, 55, 16, -29,
    28, 47, 47, -77, 0, -24, 1, 1, 38, 28, -11, 2, -55, 4, 18, 42, 99, 98, 1, 17,
    18, -21, 4, 89, 66, -32, 17, 56, 14, -2, -45, 19, -30, 26, 14, 34, -36, 5, 74,
    50, 33, 47, -37, 34, 61, -8, -62, 46, 56, -55, 0, 33, 5, -72, -29, -48, 21, 40,
    22, 3, 39, -1, 10, 32, -47, 28, 19, 92, -5, -13, 2, 12, -21, -33, -9, 31, -2,
    -25, -20, -14, 1, 53, -34, -26, 17, 72, -35, -36, -26, -86, -20, 55, -4, -53,
    -14, 47, 26, 82, -3, -41, -18, -40, -94, 87, 3, -17, 38, 54, 17, 62, -23, 61,
    20, -4, 18, 37, 21, -37, -10, -43, -32, -40, -29, 43, 75, -44, -3, 47, 9, -10,
    29, -26, 55, 35, -17, 43, 37, -8, 19, 0, -32, -49, 43, -27, 16, -81, 34, 56,
    15, -33, -13, -30, -13, -28, 54, -61, -90, -45, -101, -52, -101, 5, 22, 7, 72,
    -30, 31, 27, 42, -47, -6, -30, -30, 42, 13, -23, 63, -84, -20, -17, 61, -40,
    35, 37, 21, -8, 110, 108, 26, -49, -1, -31, 8, 10, 7, 29, -67, -29, 72, 15, 11,
    4, -34, 12, 28, -48, -21, -81, 38, -29, 26, 4, 10, 29, -11, 26, -78, -51, -52,
    27, -92, -23, -5, -11, 31, 18, -33, -49, 7, -51, -35, -57, -14, 121, -8, 29, 25,
    70, -19, 29, 48, -41, 48, -18, 19, -18, -13, 46, 27, 47, 42, 1, -33, 20, -27, 8,
    -31, 31, 1, 0, 11, -4, 32, -65, -7, 9, -11, 15, 3, -34, 42, -15, -71, -5, 3, 8,
    -8, 22, -7, -70, 10, 21, -127, -114, 13, -11, 46, -13, -10, -10, 29, -59, 43,
    -1, -17, -21, 8, -15, 12, 1, -73, -26, -5, 6, 37, 23, 46, 73, 14, -74, 84, -2,
    -22, -6, 5, -7, -26, 28, -39, -23, -22, 14, 38, 0, -2, 41, 27, -65, 30, 3, -23,
    53, 86, 35, -32, -48, -15, 32, 21, -26, -48, -26, 32, 32, 4, -70, -72, -62, -28,
    -14, -86, -10, -63, 44, -68, -41, 27, -52, 33, -56, -30, 5, 84, -54, 16, -22,
    -20, 16, 34, 14, -25, 8, -14, -13, -28, -40, 16, 41, -5, -88, -35, 55, -82, 55,
    74, -55, -12, 58, 57, -83, -26, 55, 32, -6, 42, -14, 35, -5, -36, 84, -40, -29,
    7, -20, -17, 23, -20, -49, -48, 22, 49, -30, 35, 48, 5, 34, 17, 13, 30, 33, -38,
    -37, 10, -52, -24, 67, -15, -12, -3, -11, -46, -7, 32, 10, -46, 3, 18, -7, -26,
    0, -40, 23, -46, 89, 37, 3, -29, -51, -32, 49, -51, 9, 16, -47, -26, 14, 10, 14,
    -13, 11, 16, -18, 54, -24, 18, -14, -51, -89, -24, 20, 12, 2, 62, 13, 53, -22,
    2, 22, -14, 29, -9, 51, -42, -97, 28, 49, -4, -93, -17, -26, 46, 47, 33, -33,
    25, 81, -29, 5, 17, 24, 54, -10, -14, -2, 29, 17, -4, -47, 56, 4, 9, 30, -87,
    39, -16, 39, 67, -13, 37, 13, 67, 50, -16, -55, 8, 24, -50, -1, -36, -51, -20,
    -58, 11, -28, -22, -26, 16, 7, -17, 39, -9, -21, -9, -8, -18, 37, -47, -19, 36,
    -8, 6, -39, 58, -26, -37, 11, 86, 33, 67, -35, 25, -11, -7, -22, 20, 14, 8, 8,
    7, -30, -58, 37, -1, 16, -13, 89, -6, 81, -46, -37, -7, 9, -23, -11, -41, -13,
    18, -17, -4, -42, 0, 91, -128, 33, -18, -88, -84, -11, -62, 79, -34, -39, 54,
    -17, -14, 15, 79, -33, -4, 30, 5, 8, -55, -9, -38, 10, -41, 37, -5, 2, 62, 3,
    -5, -42, 17, -50, 14, -58, -16, 26, -20, -49, 52, 73, -42, 9, 7, -50, 14, -11,
    39, 0, -45, -90, -30, -16, -19, -6, -1, 43, -7, -47, -4, 40, -6, 5, 2, 2, -20,
    -40, 39, 10, -16, 64, -11, -36, -5, 37, -16, 49, 24, -20, 17, 27, -21, -49, -49,
    -38, -19, -31, -2, 15, 52, -68, -14, 20, 38, 10, -48, -2, -52, -60, -55, -30,
    37, -32, -80, 1, -1, -12, -45, 15, 29, 8, -46, -42, -28, -38, 11, 4, 19, 2, 67,
    -44, -5, -28, 21, 17, -16, -34, 16, -6, 10, -11, 15, 2, 33, -25, -13, 8, -7, 2,
    -22, 21, -41, 10, -29, -36, 46, 19, -41, 36, -39, 10, -23, -13, -2, -53, 39,
    -25, -4]
    
    
    UBINARY Embedding from the same model (1024 dimensions = 128 packed UINT8 bytes)
    [201, 200, 65, 129, 217, 166, 185, 167, 90, 138, 0, 172, 242, 207, 165, 52, 245,
    187, 96, 215, 39, 159, 250, 126, 107, 162, 201, 123, 193, 203, 202, 123, 87, 67,
    113, 235, 253, 220, 187, 236, 220, 125, 185, 136, 102, 8, 224, 222, 220, 12, 214,
    217, 92, 16, 61, 195, 69, 220, 121, 236, 94, 136, 100, 46, 212, 250, 189, 45, 26,
    101, 20, 88, 253, 18, 51, 110, 49, 192, 37, 52, 232, 98, 204, 212, 146, 55, 249,
    32, 108, 174, 44, 237, 67, 246, 166, 29, 188, 103, 173, 230, 4, 104, 37, 79, 71,
    202, 162, 16, 160, 147, 56, 174, 82, 109, 96, 34, 230, 139, 96, 51, 129, 35, 135,
    198, 87, 42, 154, 132]
```
    

The `BINARY` vector is generated through a binary quantization mechanism using the following rule: 

  * If the `INT8` dimension value > 0, the `BINARY` dimension value is 1 
  * If the `INT8` dimension value <= 0, the `BINARY` dimension format is 0 



Consider the first 8 `INT8` dimensions from the preceding example: 
```
    [25, 11, -99, -114, 13, -17, -59, 44]
```
    

In `BINARY`, this translates to: 
```
    [1, 1, 0, 0, 1, 0, 0, 1]
```
    

Representing this as a `UINT8` byte makes it 201, which is the first byte value of the packed `UINT8` representation. So, each `BINARY` vector can therefore be inserted as a `UINT8` array whose size is: *number ofBINARYvector dimensions*/*8*. 

> **note:** `BINARY` vectors are only supported with a number of dimensions that is a multiple of 8. 

The following is an example of an invalid declaration of a `BINARY` vector column, due to the fact that the vector dimension, 12, is not divisible by 8: 
```
    CREATE TABLE vectab (id NUMBER, data VECTOR(12, BINARY));
```
    

Result:
```
    CREATE TABLE vectab (id NUMBER, data VECTOR(12, BINARY))
    *
    ERROR at line 1:
    ORA-51813: Vector of BINARY format should have a dimension count that is a multiple of 8.
```
    

The following statements are an example of a valid table creation with a `BINARY` vector column and a valid insert (string representation): 
```
    CREATE TABLE vectab(id NUMBER, data VECTOR(16, BINARY));
    INSERT INTO vectab VALUES (1, '[201, 15]');
    SELECT data FROM vectab;
```
    

Result:
```
    DATA
    ---------
    [201,15]
```
    

These next statements are examples of invalid inserts (string representation):
```
    SQL> INSERT INTO vectab VALUES (1, '[201]');
    INSERT INTO vectab VALUES (1, '[201]')
    *
    ERROR at line 1:
    ORA-51803: Vector dimension count must match the dimension count specified in
    the column definition (actual: 8, required: 16).
    
    SQL> INSERT INTO vectab VALUES (1, '[201, 15, 123]');
    INSERT INTO vectab VALUES (1, '[201, 15, 123]')
    *
    ERROR at line 1:
    ORA-51803: Vector dimension count must match the dimension count specified in
    the column definition (actual: 24, required: 16).
    
    SQL> INSERT INTO vectab VALUES (1, '[256, 15]');
    INSERT INTO vectab VALUES (1, '[256, 15]')
    *
    ERROR at line 1:
    ORA-51806: Vector column is not properly formatted (dimension value 1 is
    outside the allowed precision range).
```
    

**Parent topic:** [Create Tables Using the VECTOR Data Type](create-tables-using-vector-data-type.md)
