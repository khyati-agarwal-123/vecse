##  CREATE_PREFERENCE {#GUID-B83978CD-EAF8-4794-9652-F335C54C3385} 

Use the ` DBMS_VECTOR_CHAIN.CREATE_PREFERENCE ` preference helper procedure to create a vectorizer preference, to be used when creating or updating hybrid vector indexes. 

Purpose 

To create a vectorizer preference. 

This allows you to customize vector search parameters of a hybrid vector indexing pipeline. The goal of a vectorizer preference is to provide you with a straightforward way to configure how to chunk or embed your documents, without requiring a deep understanding of various chunking or embedding strategies. 

Usage Notes 

A **vectorizer** preference is a JSON object that collectively holds user-specified values related to the following chunking, embedding, or vector index creation parameters: 

  * Chunking ( ` UTL_TO_CHUNKS ` and ` VECTOR_CHUNKS ` ) 

  * Embedding ( ` UTL_TO_EMBEDDING ` , ` UTL_TO_EMBEDDINGS ` , and ` VECTOR_EMBEDDING ` ) 

  * Vector index creation ( ` distance ` , ` accuracy ` , and ` vector_idxtype ` ) 

All vector index preferences follow the same JSON syntax as defined for their corresponding ` DBMS_VECTOR ` and ` DBMS_VECTOR_CHAIN ` APIs. 

After creating a vectorizer preference, you can use the ` VECTORIZER ` parameter to pass this preference name in the ` paramstring ` of the ` PARAMETERS ` clause for ` CREATE_HYBRID_VECTOR_INDEX ` and ` ALTER_INDEX ` SQL statements. 

Creating a preference is optional. If you do not specify any optional preference, then the index is created with system defaults. 

Syntax 
    
        ```
    DBMS_VECTOR_CHAIN.CREATE_PREFERENCE (
        PREF_NAME     IN VARCHAR2,
        PREF_TYPE     IN VARCHAR2,
        PARAMS        IN JSON default NULL
    );
    ```

PREF_NAME 

Specify the name of the vectorizer preference to create. 

PREF_TYPE 

Type of preference. The only supported preference type is: 

` DBMS_VECTOR_CHAIN.VECTORIZER `

PARAMS 

Specify vector search-specific parameters in JSON format: 

    * [ Embedding Parameter ](create_preference.html#GUID-B83978CD-EAF8-4794-9652-F335C54C3385__P_JBQ_3NF_GDC)

    * [ Chunking Parameters ](create_preference.html#GUID-B83978CD-EAF8-4794-9652-F335C54C3385__P_VF1_JNF_GDC)

    * [ Vector Index Parameters ](create_preference.html#GUID-B83978CD-EAF8-4794-9652-F335C54C3385__P_WVP_JNF_GDC)

Embedding Parameter  : 
        
                ```
        { "model" :  }
        ```

For example: 
        
                ```
        { "model" : MY_INDB_MODEL }
        ```

` model ` specifies the name under which your ONNX embedding model is stored in the database. 

If you do not have an in-database embedding model in ONNX format, then perform the steps listed in [ *Oracle Database AI Vector Search User's Guide*  ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6) . 

Chunking Parameters  : 
        
                ```
        {
            "by"           :  mode,
            "max"          :  max,
            "overlap"      :  overlap,
            "split"        :  split_condition,
            "vocabulary"   :  vocabulary_name,
            "language"     :  nls_language,
            "normalize"    :  normalize_mode,
            "extended"     :  boolean
        }
        ```

For example: 
        
                ```
        JSON(
         '{ "by"           :    "vocabulary",
            "max"          :    "100",
            "overlap"      :    "0",
            "split"        :    "none",
            "vocabulary"   :    "myvocab",
            "language"     :    "american",
            "normalize"    :    "all" 
          }')
        ```

If you specify ` split ` as ` custom ` and ` normalize ` as ` options ` , then you must additionally specify the ` custom_list ` and ` norm_options ` parameters, respectively: 
        
                ```
        JSON(
         '{ "by"           :    "vocabulary",
            "max"          :    "100",
            "overlap"      :    "0",
            "split"        :    "custom",
            "custom_list"  :    [ "" , "" ],
            "vocabulary"   :    "myvocab",
            "language"     :    "american",
            "normalize"    :    "options",
            "norm_options" :    [ "whitespace" ] 
          }')
        ```

The following table describes all the chunking parameters: 

Parameter  |  Description and Acceptable Values   
---|---  
` by ` |  Specify a mode for splitting your data, that is, to split by counting the number of characters, words, or vocabulary tokens.  Valid values  :  <li>
      * ` characters ` (or ` chars ` ):  Splits by counting the number of characters.  </li> <li>
      * ` words ` :  Splits by counting the number of words.  Words are defined as sequences of alphabetic characters, sequences of digits, individual punctuation marks, or symbols. For segmented languages without whitespace word boundaries (such as Chinese, Japanese, or Thai), each native character is considered a word (that is, unigram).  </li> <li>
      * ` vocabulary ` :  Splits by counting the number of vocabulary tokens.  Vocabulary tokens are words or word pieces, recognized by the vocabulary of the tokenizer that your embedding model uses. You can load your vocabulary file using the chunker helper API ` DBMS_VECTOR_CHAIN.CREATE_VOCABULARY ` .  Note  : For accurate results, ensure that the chosen model matches the vocabulary file used for chunking. If you are not using a vocabulary file, then ensure that the input length is defined within the token limits of your model.  </li> Default value  : ` words `  
` max ` |  Specify a limit on the maximum size of each chunk. This setting splits the input text at a fixed point where the maximum limit occurs in the larger text. The units of ` max ` correspond to the ` by ` mode, that is, to split data when it reaches the maximum size limit of a certain number of characters, words, numbers, punctuation marks, or vocabulary tokens.  Valid values  :  <li>
        * ` by characters ` : ` 50 ` to ` 4000 ` characters  </li> <li>
        * ` by words ` : ` 10 ` to ` 1000 ` words  </li> <li>
        * ` by vocabulary ` : ` 10 ` to ` 1000 ` tokens  </li> Default value  : ` 100 `  
` split [by] ` |  Specify where to split the input text when it reaches the maximum size limit. This helps to keep related data together by defining appropriate boundaries for chunks.  Valid values  :  <li>
          * ` none ` :  Splits at the ` max ` limit of characters, words, or vocabulary tokens.  </li> <li>
          * ` newline ` , ` blankline ` , and ` space ` :  These are single-split character conditions that split at the last split character before the ` max ` value.  Use ` newline ` to split at the end of a line of text. Use ` blankline ` to split at the end of a blank line (sequence of characters, such as two newlines). Use ` space ` to split at the end of a blank space.  </li> <li>
          * ` recursively ` :  This is a multiple-split character condition that breaks the input text using an ordered list of characters (or sequences).  ` recursively ` is predefined as ` BLANKLINE ` , ` newline ` , ` space ` , ` none ` in this order:  1\. If the input text is more than the ` max ` value, then split by the first split character.  2\. If that fails, then split by the second split character.  3\. And so on.  4\. If no split characters exist, then split by ` max ` wherever it appears in the text.  </li> <li>
          * ` sentence ` :  This is an end-of-sentence split condition that breaks the input text at a sentence boundary.  This condition automatically determines sentence boundaries by using knowledge of the input language's sentence punctuation and contextual rules. This language-specific condition relies mostly on end-of-sentence (EOS) punctuations and common abbreviations.  Contextual rules are based on word information, so this condition is only valid when splitting the text by words or vocabulary (not by characters).  Note  : This condition obeys the ` by word ` and ` max ` settings, and thus may not determine accurate sentence boundaries in some cases. For example, when a sentence is larger than the ` max ` value, it splits the sentence at ` max ` . Similarly, it includes multiple sentences in the text only when they fit within the ` max ` limit.  </li> <li>
          * ` custom ` :  Splits based on a custom split characters list. You can provide custom sequences up to a limit of ` 16 ` split character strings, with a maximum length of ` 10 ` each.  Specify an array of valid text literals using the ` custom_list ` parameter. 
                    
                                        ```
                    {
                        "split"        :  "custom",
                        "custom_list"  :  [ "split_chars1", ... ]
                    }
                    ```

For example: 
                    
                                        ```
                    {
                        "split"        :    "custom",
                        "custom_list"  :    [ "" , "" ]
                    }
                    ```

Note  : You can omit sequences only for tab ( ` \t ` ), newline ( ` \n ` ), and linefeed ( ` \r ` ).  </li> Default value  : ` recursively `  
` overlap ` |  Specify the amount (as a positive integer literal or zero) of the preceding text that the chunk should contain, if any. This helps in logically splitting up related text (such as a sentence) by including some amount of the preceding chunk text.  The amount of overlap depends on how the maximum size of the chunk is measured (in characters, words, or vocabulary tokens). The overlap begins at the specified ` split ` condition (for example, at ` newline ` ).  Valid value  : ` 5% ` to ` 20% ` of ` max ` Default value  : ` 0 `  
` language ` |  Specify the language of your input data.  This clause is important, especially when your text contains certain characters (for example, punctuations or abbreviations) that may be interpreted differently in another language.  Valid values  :  <li>
            * NLS-supported language name or its abbreviation, as listed in [ *Oracle Database Globalization Support Guide*  ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=NLSPG-GUID-D2FCFD55-EDC3-473F-9832-AAB564457830) .  </li> <li>
            * Custom language name or its abbreviation, as listed in [ Supported Languages and Data File Locations ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-8C8AAE2F-E64A-470F-B109-BE1AC2D6E498) . You use the ` DBMS_VECTOR_CHAIN.CREATE_LANG_DATA ` chunker helper API to load language-specific data (abbreviation tokens) into the database, for your specified language.  </li> Default value  : ` NLS_LANGUAGE ` from session   
` normalize ` |  Automatically pre-processes or post-processes issues (such as multiple consecutive spaces and smart quotes) that may arise when documents are converted into text. Oracle recommends you to use a normalization mode to extract high-quality chunks.  Valid values  :  <li>
              * ` none ` :  Applies no normalization.  </li> <li>
              * ` all ` :  Normalizes common multi-byte (unicode) punctuation to standard single-byte.  </li> <li>
              * ` options ` :  Specify an array of normalization options using the ` norm_options ` parameter. 
                            
                                                        ```
                            {
                                "normalize"    :  "options",
                                "norm_options" :  [ "normalize_option1", ... ]
                            }
                            ```

<li>
                * ` punctuation ` :  Converts quotes, dashes, and other punctuation characters supported in the character set of the text to their common ASCII form. For example:  <li>
                  * U+2013 (En Dash) maps to U+002D (Hyphen-Minus)  </li> <li>
                  * U+2018 (Left Single Quotation Mark) maps to U+0027 (Apostrophe)  </li> <li>
                  * U+2019 (Right Single Quotation Mark) maps to U+0027 (Apostrophe)  </li> <li>
                  * U+201B (Single High-Reversed-9 Quotation Mark) maps to U+0027 (Apostrophe)  </li> </li> <li>
                  * ` whitespace ` :  Minimizes whitespace by eliminating unnecessary characters.  For example, retain blanklines, but remove any extra newlines and interspersed spaces or tabs: ` " \n \n " => "\n\n" ` </li> <li>
                  * ` widechar ` :  Normalizes wide, multi-byte digits and (a-z) letters to single-byte.  These are multi-byte equivalents for ` 0-9 ` and ` a-z A-Z ` , which can show up in Chinese, Japanese, or Korean text.  </li> For example: 
                                    
                                                                        ```
                                    {
                                        "normalize"    :  "options",
                                        "norm_options" :  [ "whitespace" ]
                                    }
                                    ```

</li> Default value  : ` none `  
` extended ` |  Increases the output limit of a ` VARCHAR2 ` string to ` 32767 ` bytes, without requiring you to set the ` max_string_size ` parameter to ` extended ` .  Default value  : ` 4000 ` or ` 32767 ` (when ` max_string_size=extended ` )   
  
Vector Index Parameters  : 
                                    
                                                                        ```
                                    { 
                                      "distance"        :  ,
                                      "accuracy"        :  , 
                                      "vector_idxtype"  :  
                                    }
                                    ```

For example: 
                                    
                                                                        ```
                                    { 
                                      "distance"        :  COSINE,
                                      "accuracy"        :  95, 
                                      "vector_idxtype"  :  HNSW
                                    }
                                    ```

Parameter  |  Description   
---|---  
` distance ` |  Distance metric or mathematical function used to compute the distance between vectors:  <li>
                    * ` COSINE ` </li> <li>
                    * ` MANHATTAN ` </li> <li>
                    * ` DOT ` </li> <li>
                    * ` EUCLIDEAN ` </li> <li>
                    * ` L2_SQUARED ` </li> <li>
                    * ` EUCLIDEAN_SQUARED ` </li> Note  : Currently, the ` HAMMING ` and ` JACCARD ` vector distance metrics are not supported with hybrid vector indexes.  For detailed information on each of these metrics, see [ Vector Distance Functions and Operators ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-8667D062-8084-4E55-8BD7-2C84E05F52FA) .  Default value  : ` COSINE `  
` accuracy ` |  Target accuracy at which the approximate search should be performed when running an approximate search query using vector indexes.  As explained in [ Understand Approximate Similarity Search Using Vector Indexes ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-69B32E29-4ACA-4483-8607-BEDC2EFF6EEB) , you can specify non-default target accuracy values either by specifying a percentage value or by specifying internal parameters values, depending on the index type you are using.  <li>
                      * For a Hierarchical Navigable Small World (HNSW) approximate search:  In the case of an HNSW approximate search, you can specify a target accuracy percentage value to influence the number of candidates considered to probe the search. This is automatically calculated by the algorithm. A value of 100 will tend to impose a similar result as an exact search, although the system may still use the index and will not perform an exact search. The optimizer may choose to still use an index as it may be faster to do so given the predicates in the query. Instead of specifying a target accuracy percentage value, you can specify the ` EFSEARCH ` parameter to impose a certain maximum number of candidates to be considered while probing the index. The higher that number, the higher the accuracy.  For detailed information, see [ Understand Hierarchical Navigable Small World Indexes ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-23E74FFC-B6A5-4374-9813-F2634AC43410) .  </li> <li>
                      * For an Inverted File Flat (IVF) approximate search:  In the case of an IVF approximate search, you can specify a target accuracy percentage value to influence the number of partitions used to probe the search. This is automatically calculated by the algorithm. A value of 100 will tend to impose an exact search, although the system may still use the index and will not perform an exact search. The optimizer may choose to still use an index as it may be faster to do so given the predicates in the query. Instead of specifying a target accuracy percentage value, you can specify the ` NEIGHBOR PARTITION PROBES ` parameter to impose a certain maximum number of partitions to be probed by the search. The higher that number, the higher the accuracy.  For detailed information, see [ Understand Inverted File Flat Vector Indexes ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-332E5EA7-6CE9-4392-AB2B-3124BF832C5E) .  </li> Valid range for both HNSW and IVF vector indexes is:  ` > 0 and <= 100 ` Default value  : None   
` vector_idxtype ` |  Type of vector index to create:  <li>
                        * ` HNSW ` for an HNSW vector index  </li> <li>
                        * ` IVF ` for an IVF vector index  </li> For detailed information on each of these index types, see [ Manage the Different Categories of Vector Indexes ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-5D9B6B92-C62C-4927-9FB2-7A4437F24A19) .  Default value  : ` IVF `  
  
Example 
                                                
                                                                                                ```
                                                begin
                                                  DBMS_VECTOR_CHAIN.CREATE_PREFERENCE(
                                                    'my_vec_spec',
                                                     DBMS_VECTOR_CHAIN.VECTORIZER,
                                                     json('{ "vector_idxtype" :  "hnsw",
                                                             "model"          :  "my_doc_model",
                                                             "by"             :  "words",
                                                             "max"            :  100,
                                                             "overlap"        :  10,
                                                             "split"          :  "recursively,
                                                             "language"       :  "american"  
                                                              }'));
                                                end;
                                                /
                                                
                                                CREATE HYBRID VECTOR INDEX my_hybrid_idx on 
                                                    doc_table(text_column) 
                                                    parameters('VECTORIZER my_vec_spec');
                                                ```

**Related Topics**

                          * [ CREATE HYBRID VECTOR INDEX ](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=CCREF-GUID-22B07106-419E-4C55-B4E1-3FD691E033BC)

**Parent topic:** [ DBMS_VECTOR_CHAIN ](dbms_vector_chain-vecse.html)