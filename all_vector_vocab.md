## ALL_VECTOR_VOCAB {#GUID-279A65A4-AFA0-4EEA-9134-5319E4739477}

The `ALL_VECTOR_VOCAB` view displays all custom token vocabularies. 

Column Name | Data Type | Description  
---|---|---  
`VOCAB_OWNER` |  `VARCHAR2(128)` |  Owner of the vocabulary (for example, `SYS`)   
`VOCAB_NAME` |  `VARCHAR2(128)` |  User-specified name of the vocabulary (for example, `DOC_VOCAB`)   
`FORMAT` |  `VARCHAR2(4)` |  Format of the vocabulary, such as `XLM`, `BERT`, or `GPT2`  
`CASED` |  `VARCHAR2(7)` |  Character-casing of the vocabulary, that is, vocabulary to be treated as cased or uncased  
  
**Parent topic:** [Text Processing Views](text-processing-views.md)
