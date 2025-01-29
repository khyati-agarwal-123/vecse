## USER_VECTOR_VOCAB {#GUID-BE37A777-B5CA-4CCB-BAA1-A0DED65C0D0C}

The `USER_VECTOR_VOCAB` view displays all custom token vocabularies created by the current user. 

Column Name | Data Type | Description  
---|---|---  
`VOCAB_NAME` |  `VARCHAR2(128)` |  User-specified name of the vocabulary (for example, `DOC_VOCAB`)   
`FORMAT` |  `VARCHAR2(4)` |  Format of the vocabulary, such as `XLM`, `BERT`, or `GPT2`  
`CASED` |  `VARCHAR2(7)` |  Character-casing of the vocabulary, that is, vocabulary to be treated as cased or uncased  
  
**Parent topic:** [Text Processing Views](text-processing-views.md)
