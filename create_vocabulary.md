## CREATE_VOCABULARY {#GUID-2D19528E-0F0D-4102-8EC7-E9EA62C66C2D}

Use the `DBMS_VECTOR_CHAIN.CREATE_VOCABULARY` chunker helper procedure to load your own token vocabulary file into the database. 

Purpose

To create custom token vocabulary that is recognized by the tokenizer used by your vector embedding model.

A vocabulary contains a set of tokens (words and word pieces) that are collected during a model's statistical training process. You can supply this data to the chunker to help in accurately selecting the text size that approximates the maximum input limit imposed by the tokenizer of your embedding model. 

Usage Notes

  * Usually, the supported vocabulary files (containing recognized tokens) are included as part of a model's distribution. Oracle recommends to use the vocabulary files associated with your model. 

If a vocabulary file is not available, then you may download one of the following files depending on the tokenizer type: 
    * **WordPiece**: 

Vocabulary file (`vocab.txt`) for the "bert-base-uncased" (English) or "bert-base-multilingual-cased" model 

    * **Byte-Pair Encoding (BPE)**: 

Vocabulary file (`vocab.json`) for the "GPT2" model 

Use the following python script to extract the file:
```
        import json
        import sys
        
        with open(sys.argv[1], encoding="utf-8") as f:
        d = json.load(f)
        for term in d:
        print(term)
```
        

    * **SentencePiece**: 

Vocabulary file (`tokenizer.json`) for the "xlm-roberta-base" model 

Use the following python script to extract the file:
```
        import json
        import sys
        
        with open(sys.argv[1], encoding="utf-8") as f:
        d = json.load(f)
        for entry in d["model"]["vocab"]:
        print(entry[0])
```
        

Ensure to save your vocabulary files in `UTF-8` encoding. 

  * You can create a vocabulary based on the tokens loaded in the `schema.table.column`, using a user-specified vocabulary name (`vocabulary_name`). 

After loading your vocabulary data, you can use the `by vocabulary` chunking mode (with `VECTOR_CHUNKS` or `UTL_TO_CHUNKS`) to split input data by counting the number of tokens. 

  * You can query these data dictionary views to access existing vocabulary data: 
    * `ALL_VECTOR_VOCAB` displays all available vocabularies. 

    * `USER_VECTOR_VOCAB` displays vocabularies from the schema of the current user. 

    * `ALL_VECTOR_VOCAB_TOKENS` displays a list of tokens from all available vocabularies. 

    * `USER_VECTOR_VOCAB_TOKENS` displays a list of tokens from the vocabularies owned by the current user. 




Syntax
```
    DBMS_VECTOR_CHAIN.CREATE_VOCABULARY(
    PARAMS      IN JSON default NULL
    );
```
    

PARAMS

Specify the input parameters in JSON format:
```
    {
    table_name,
    column_name,
    vocabulary_name,
    format,
    cased
    }
```
    

**Table: Parameter Details**

Parameter | Description | Required | Default Value  
---|---|---|---  
`table_name` |  Name of the table (along with the optional table owner) in which you want to load the vocabulary file |  Yes |  No value  
`column_name` |  Column name in the vocabulary table in which you want to load the vocabulary file |  Yes |  No value  
`vocabulary_name` |  User-specified name of the vocabulary, along with the optional owner name (if other than the current owner) |  Yes |  No value  
`format` | 

  * `xlm` for SentencePiece tokenization  <br>`bert` for WordPiece tokenization  <br>`gpt2` for BPE tokenization 

|  Yes |  No value  
`cased` |  Character-casing of the vocabulary, that is, vocabulary to be treated as cased or uncased |  No |  `false`  
  
Example
```
    DECLARE
    params clob := '{"table_name"       : "doc_vocabtab",
    "column_name"      : "token",
    "vocabulary_name"  : "doc_vocab",
    "format"           : "bert",
    "cased"            : false}';
    
    BEGIN
    dbms_vector_chain.create_vocabulary(json(params));
    END;
    /
```
    

**End-to-end example**: 

To run an end-to-end example scenario using this procedure, see [Create and Use Custom Vocabulary](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-B6527DDC-8EF0-479E-8965-6C2459E7827A). 

**Related Topics**

  * [VECTOR_CHUNKS](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-5927E2FA-6419-4744-A7CB-3E62DBB027AD)
  * [UTL_TO_CHUNKS](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-4E145629-7098-4C7C-804F-FC85D1F24240)
  * [Text Processing Views](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-E2B9F02C-E2A6-439B-9A2E-177FF7FA6EE0)



**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
