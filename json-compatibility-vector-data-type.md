## JSON Compatibility with the VECTOR Data Type {#GUID-AEEFF7F7-BC36-4446-B67D-A51C210CA6CB}

The JSON data type supports the `VECTOR` type as a JSON scalar type. A `VECTOR` instance is convertible to a JSON type instance and vice versa using the JSON constructor and the `VECTOR` constructor, respectively. 

Note that when converting a JSON array to a `VECTOR` type instance, the numbers in the JSON array do not have to be of the same numeric type. The input data types do not change the default numeric type for the vector and the `VECTOR` constructor includes an argument to set the format. Using the `VECTOR` constructor (or `TO_VECTOR`) with a JSON data type instance will succeed as long as there is no conversion error. 

The following SQL/JSON functions and item methods are compatible for use with the `VECTOR` data type: 

  * `JSON_VALUE`
  * `JSON_OBJECT`
  * `JSON_ARRAY`
  * `JSON_TABLE`
  * `JSON_SCALAR`
  * `JSON_TRANSFORM`
  * `JSON_SERIALIZE`
  * `type()`
  * `.vector()`
  * `.string()`
  * `.stringify()`



> **note:** See Also: 

  * [*Oracle Database JSON Developer’s Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ADJSN-GUID-E075CA14-DC89-4F9D-AD6F-C351F774575A) for more information about `VECTOR` as a data type for JSON data 
  * [*Oracle Database JSON Developer’s Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=ADJSN-GUID-8656CAB9-C293-4A99-BB62-F38F3CFC4C13) for information about SQL/JSON path expression item methods, including `vector()`



**Parent topic:** [Use SQL Functions for Vector Operations](use-sql-functions-vector-operations.md)
