##  V$VECTOR_INDEX {#GUID-183E165B-9155-41F7-9FE2-160D6BF73849} 

This fixed view provides diagnostic information about vector indexes and is available with Autonomous Database Serverless (ADB-S). 

Column Name  |  Data Type  |  Description   
---|---|---  
` OWNER ` |  ` VARCHAR(129) ` |  User name of the vector index owner   
` INDEX_NAME ` |  ` VARCHAR(129) ` |  Name of the vector index   
` PARTITION_NAME ` |  ` VARCHAR(129) ` |  Object partition name (set to ` NULL ` for non-partitioned objects)   
` INDEX_ORGANIZATION ` |  ` VARCHAR(129) ` |  The vector index organization. The value can be one of the following: 

<li>
  * ` NEIGHBOR PARTITIONS ` </li> <li>
  * ` INMEMORY NEIGHBOR GRAPH ` </li>  
` INDEX_OBJN ` |  ` NUMBER ` |  Index object number   
` ALLOCATED_BYTES ` |  ` NUMBER ` |  Total amount of memory allocated to this vector index, in bytes.  When the vector index is an IVF index, the value is ` 0 ` .   
` USED_BYTES ` |  ` NUMBER ` |  Total amount of memory currently used by the vector index, in bytes.  When the vector index is an IVF index, the value is ` 0 ` .   
` NUM_VECTORS ` |  ` NUMBER ` |  The number of vectors currently indexed in the main-memory storage of the index as of the latest snapshot   
` NUM_REPOP ` |  ` NUMBER ` |  Number of times this index has been repopulated   
` INDEX_USED_COUNT ` |  ` NUMBER ` |  Number of times this vector index has been used by queries   
` DISTANCE_TYPE ` |  ` VARCHAR2(129) ` |  Possible distance values:  <li>
    * ` EUCLIDEAN ` </li> <li>
    * ` EUCLIDEAN_SQUARED ` </li> <li>
    * ` COSINE ` </li> <li>
    * ` DOT ` </li> <li>
    * ` MANHATTAN ` </li> <li>
    * ` HAMMING ` </li>  
` INDEX_DIMENSIONS ` |  ` NUMBER ` |  Number of dimensions of the indexed vector   
` INDEX_DIM_TYPE ` |  ` VARCHAR2(129) ` |  Type of the dimensions of the index vector.  Possible values:  <li>
      * ` FLOAT16 ` </li> <li>
      * ` FLOAT32 ` </li> <li>
      * ` FLOAT64 ` </li> <li>
      * ` INT8 ` </li>  
` DEFAULT_ACCURACY ` |  ` NUMBER ` |  The accuracy to achieve when performing approximate search on this vector index if a target query accuracy is not provided   
` CON_ID ` |  ` NUMBER ` |  The ID of the container to which the data pertains. Possible values include the following:  <li>
        * ` 0 ` : This value is used for rows containing data that pertain to the entire multitenant container database (CDB). This value is also used for rows in non-CDBs. </li> <li>
        * ` 1 ` : This value is used for rows containing data that pertain to only the root. </li> <li>
        * ` n ` : Where ` n ` is the applicable container ID for the rows containing data. </li>  
  
**Parent topic:** [ Vector Index and Hybrid Vector Index Views ](vector-index-and-hybrid-vector-index-views.html)