## UTL_TO_CHUNKS {#GUID-4E145629-7098-4C7C-804F-FC85D1F24240}

Use the `DBMS_VECTOR_CHAIN.UTL_TO_CHUNKS` chainable utility function to split a large plain text document into smaller chunks of text. 

Purpose

To perform a text-to-chunks transformation. This chainable utility function internally calls the `VECTOR_CHUNKS` SQL function for the operation. 

To embed a large document, you may first need to split it into multiple appropriate-sized segments or chunks through a splitting process known as chunking (as explained in [Understand the Stages of Data Transformations](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-D6F1E7B6-5642-46A2-B84D-B8E37E4C353E)). A chunk can be words (to capture specific words or word pieces), sentences (to capture a specific meaning), or paragraphs (to capture broader themes). A single document may be split into multiple chunks, each transformed into a vector. 

Syntax
```
    DBMS_VECTOR_CHAIN.UTL_TO_CHUNKS (
    DATA         IN  CLOB | VARCHAR2,
    PARAMS       IN  JSON  default  NULL
    ) return VECTOR_ARRAY_T;
```
    

DATA

This function accepts the input data type as `CLOB` or `VARCHAR2`. 

It returns an array of `CLOB`s, where each `CLOB` contains a chunk along with its metadata in JSON format, as follows: 
```
    {
    "chunk_id"     : NUMBER,
    "chunk_offset" : NUMBER,
    "chunk_length" : NUMBER,
    "chunk_data"   : "VARCHAR2(4000)"
    }
```
    

For example:
```
    {"chunk_id":1,"chunk_offset":1,"chunk_length":6,"chunk_data":"sample"}
```
    

Where, 

  * `chunk_id` specifies the chunk ID for each chunk. 

  * `chunk_offset` specifies the original position of each chunk in the source document, relative to the start of document which has a position of `1`. 

  * `chunk_length` specifies the character length of each chunk. 

  * `chunk_data` displays text pieces from each chunk. 




PARAMS

Specify input parameters in JSON format.
```
    {
    "by"           :     mode,
    "max"          :     max,
    "overlap"      :     overlap,
    "split"        :     split_condition,
    "custom_list"  :     [ split_chars1, ... ],
    "vocabulary"   :     vocabulary_name,
    "language"     :     nls_language,
    "normalize"    :     normalize_mode,
    "norm_options" :     [ normalize_option1, ... ],
    "extended"     :     boolean
    }
```
    

For example:
```
    JSON('
    { "by"           :    "vocabulary",
    "vocabulary"   :    "myvocab",
    "max"          :    "100",
    "overlap"      :    "0",
    "split"        :    "custom",
    "custom_list"  :    [ "" , "" ],
    "language"     :    "american",
    "normalize"    :    "options",
    "norm_options" :    [ "whitespace" ]
    }')
```
    

Here is a complete description of these parameters:

Parameter | Description and Acceptable Values  
---|---  
`by` |  Specify a mode for splitting your data, that is, to split by counting the number of characters, words, or vocabulary tokens.<br>**Valid values**: <br>* `characters` (or `chars`): <br>Splits by counting the number of characters. <br>`words`: <br>Splits by counting the number of words.<br>Words are defined as sequences of alphabetic characters, sequences of digits, individual punctuation marks, or symbols. For segmented languages without whitespace word boundaries (such as Chinese, Japanese, or Thai), each native character is considered a word (that is, unigram). <br>`vocabulary`: <br>Splits by counting the number of vocabulary tokens.<br>Vocabulary tokens are words or word pieces, recognized by the vocabulary of the tokenizer that your embedding model uses. You can load your vocabulary file using the chunker helper API `DBMS_VECTOR_CHAIN.CREATE_VOCABULARY`. <br>**Note**: <br>For accurate results, ensure that the chosen model matches the vocabulary file used for chunking. If you are not using a vocabulary file, then ensure that the input length is defined within the token limits of your model. 
<br>**Default value**: <br>`words`  
`max` |  Specify a limit on the maximum size of each chunk. This setting splits the input text at a fixed point where the maximum limit occurs in the larger text. The units of `max` correspond to the `by` mode, that is, to split data when it reaches the maximum size limit of a certain number of characters, words, numbers, punctuation marks, or vocabulary tokens. <br>**Valid values**: <br>* `by characters`: <br>`50` to `4000` characters  <br>`by words`: <br>`10` to `1000` words  <br>`by vocabulary`: <br>`10` to `1000` tokens 
<br>**Default value**: <br>`100`  
`split [by]` |  Specify where to split the input text when it reaches the maximum size limit. This helps to keep related data together by defining appropriate boundaries for chunks. <br>**Valid values**: <br>* `none`: <br>Splits at the `max` limit of characters, words, or vocabulary tokens.  <br>`newline`, `blankline`, and `space`: <br>These are single-split character conditions that split at the last split character before the `max` value. <br>Use `newline` to split at the end of a line of text. Use `blankline` to split at the end of a blank line (sequence of characters, such as two newlines). Use `space` to split at the end of a blank space.  <br>`recursively`: <br>This is a multiple-split character condition that breaks the input text using an ordered list of characters (or sequences). <br>`recursively` is predefined as `BLANKLINE`, `newline`, `space`, `none` in this order: <br>1\. If the input text is more than the `max` value, then split by the first split character. <br>2\. If that fails, then split by the second split character. <br>3\. And so on. <br>4\. If no split characters exist, then split by `max` wherever it appears in the text.  <br>`sentence`: <br>This is an end-of-sentence split condition that breaks the input text at a sentence boundary.<br>This condition automatically determines sentence boundaries by using knowledge of the input language's sentence punctuation and contextual rules. This language-specific condition relies mostly on end-of-sentence (EOS) punctuations and common abbreviations. <br>Contextual rules are based on word information, so this condition is only valid when splitting the text by words or vocabulary (not by characters). <br>**Note**: <br>This condition obeys the `by word` and `max` settings, and thus may not determine accurate sentence boundaries in some cases. For example, when a sentence is larger than the `max` value, it splits the sentence at `max`. Similarly, it includes multiple sentences in the text only when they fit within the `max` limit.  <br>`custom`: <br>Splits based on a custom split characters list. You can provide custom sequences up to a limit of `16` split character strings, with a maximum length of `10` each. <br>Specify an array of valid text literals using the `custom_list` parameter. <br>```{ "split" : <br>"custom", "custom_list" : <br>[ "split_chars1", ... ] }```<br>For example: <br>```{ "split" : <br>"custom", "custom_list" : <br>[ "" , "" ] }```<br>**Note**: <br>You can omit sequences only for tab (`\t`), newline (`\n`), and linefeed (`\r`). 
<br>**Default value**: <br>`recursively`  
`overlap` |  Specify the amount (as a positive integer literal or zero) of the preceding text that the chunk should contain, if any. This helps in logically splitting up related text (such as a sentence) by including some amount of the preceding chunk text. <br>The amount of overlap depends on how the maximum size of the chunk is measured (in characters, words, or vocabulary tokens). The overlap begins at the specified `split` condition (for example, at `newline`). <br>**Valid value**: <br>`5%` to `20%` of `max`<br>**Default value**: <br>`0`  
`language` |  Specify the language of your input data.<br>This clause is important, especially when your text contains certain characters (for example, punctuations or abbreviations) that may be interpreted differently in another language.<br>**Valid values**: <br>* NLS-supported language name or its abbreviation, as listed in [*Oracle Database Globalization Support Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=NLSPG-GUID-D2FCFD55-EDC3-473F-9832-AAB564457830).  <br>Custom language name or its abbreviation, as listed in [Supported Languages and Data File Locations](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-8C8AAE2F-E64A-470F-B109-BE1AC2D6E498). You use the `DBMS_VECTOR_CHAIN.CREATE_LANG_DATA` chunker helper API to load language-specific data (abbreviation tokens) into the database, for your specified language. 
<br>**Note**: <br>You must use escape characters with any language abbreviation that is also a SQL reserved word (for example, language abbreviations such as `IN`, `AS`, `OR`, `IS`). <br>For example: <br>```SELECT dbms_vector_chain.utl_to_chunks('this is an example', JSON('{ "language" : <br>"**\"in\"**" }')) from dual;```<br>```SELECT dbms_vector_chain.utl_to_chunks('this is an example', JSON_OBJECT('language' value '**"in"**' RETURNING JSON)) from dual;```<br>**Default value**: <br>`NLS_LANGUAGE` from session   
`normalize` |  Automatically pre-processes or post-processes issues (such as multiple consecutive spaces and smart quotes) that may arise when documents are converted into text. Oracle recommends you to use a normalization mode to extract high-quality chunks.<br>**Valid values**: <br>* `none`: <br>Applies no normalization. <br>`all`: <br>Normalizes common multi-byte (unicode) punctuation to standard single-byte. <br>`options`: <br>Specify an array of normalization options using the `norm_options` parameter. <br>```{ "normalize" : <br>"options", "norm_options" : <br>[ "normalize_option1", ... ] }```<br>* `punctuation`: <br>Converts quotes, dashes, and other punctuation characters supported in the character set of the text to their common ASCII form. For example: <br>* U+2013 (En Dash) maps to U+002D (Hyphen-Minus)  <br>U+2018 (Left Single Quotation Mark) maps to U+0027 (Apostrophe)  <br>U+2019 (Right Single Quotation Mark) maps to U+0027 (Apostrophe)  <br>U+201B (Single High-Reversed-9 Quotation Mark) maps to U+0027 (Apostrophe)

<br>`whitespace`: <br>Minimizes whitespace by eliminating unnecessary characters. <br>For example, retain blanklines, but remove any extra newlines and interspersed spaces or tabs: <br>`" \n \n " => "\n\n"` <br>`widechar`: <br>Normalizes wide, multi-byte digits and (a-z) letters to single-byte.<br>These are multi-byte equivalents for `0-9` and `a-z A-Z`, which can show up in Chinese, Japanese, or Korean text. 
<br>For example: <br>```{ "normalize" : <br>"options", "norm_options" : <br>[ "whitespace" ] }```<br>**Default value**: <br>`none`  
`extended` |  Increases the output limit of a `VARCHAR2` string to `32767` bytes, without requiring you to set the `max_string_size` parameter to `extended`. <br>**Default value**: <br>`4000` or `32767` (when `max_string_size=extended`)   
  
Example
```
    SELECT D.id doc,
    JSON_VALUE(C.column_value, '$.chunk_id' RETURNING NUMBER) AS id,
    JSON_VALUE(C.column_value, '$.chunk_offset' RETURNING NUMBER) AS pos,
    JSON_VALUE(C.column_value, '$.chunk_length' RETURNING NUMBER) AS siz,
    JSON_VALUE(C.column_value, '$.chunk_data') AS txt
    FROM docs D,
    dbms_vector_chain.utl_to_chunks(D.text,
    JSON('{ "by"       : "words",
    "max"      : "100",
    "overlap"  : "0",
    "split"    : "recursively",
    "language" : "american",
    "normalize": "all" }')) C;
```
    

**End-to-end examples**: 

To run end-to-end example scenarios using this function, see [Perform Chunking With Embedding](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-9CB40305-392A-4CB0-AD49-7730504BEC18) and [Configure Chunking Parameters](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-DA2DE806-CF83-4D5A-977B-B405FC9715CF). 

**Related Topics**

  * [VECTOR_CHUNKS](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-5927E2FA-6419-4744-A7CB-3E62DBB027AD)



**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
