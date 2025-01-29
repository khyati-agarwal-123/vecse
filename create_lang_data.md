## CREATE_LANG_DATA {#GUID-C9756FA9-B0B6-4750-8D9C-ADEF8B67C675}

Use the `DBMS_VECTOR_CHAIN.CREATE_LANG_DATA` chunker helper procedure to load your own language data file into the database. 

Purpose

To create custom language data for your chosen language (specified using the `language` chunking parameter). 

A language data file contains language-specific abbreviation tokens. You can supply this data to the chunker to help in accurately determining sentence boundaries of chunks, by using knowledge of the input language's end-of-sentence (EOS) punctuations, abbreviations, and contextual rules.

Usage Notes

  * All supported languages are distributed with the default language-specific abbreviation dictionaries. You can create a language data based on the abbreviation tokens loaded in the `schema.table.column`, using a user-specified language data name (`PREFERENCE_NAME`). 

  * After loading your language data, you can use language-specific chunking by specifying the `language` chunking parameter with `VECTOR_CHUNKS` or `UTL_TO_CHUNKS`. 

  * You can query these data dictionary views to access existing language data: 
    * `ALL_VECTOR_LANG` displays all available languages data. 

    * `USER_VECTOR_LANG` displays languages data from the schema of the current user. 

    * `ALL_VECTOR_ABBREV_TOKENS` displays abbreviation tokens from all available language data. 

    * `USER_VECTOR_ABBREV_TOKENS` displays abbreviation tokens from the language data owned by the current user. 




Syntax
```
    DBMS_VECTOR_CHAIN.CREATE_LANG_DATA (
    PARAMS       IN JSON default NULL
    );
```
    

PARAMS

Specify the input parameters in JSON format:
```
    {
    table_name,
    column_name,
    language,
    preference_name
    }
```
    

**Table: Parameter Details**

Parameter | Description | Required | Default Value  
---|---|---|---  
`table_name` |  Name of the table (along with the optional table owner) in which you want to load the language data |  Yes |  No value  
`column_name` |  Column name in the language data table in which you want to load the language data |  Yes |  No value  
`language` |  Any supported language name, as listed in [Supported Languages and Data File Locations](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-8C8AAE2F-E64A-470F-B109-BE1AC2D6E498) |  Yes |  No value  
`preference_name` |  User-specified preference name for this language data |  Yes |  No value  
  
Example
```
    declare
    params CLOB := '{"table_name"      : "eos_data_1",
    "column_name"     : "token",
    "language"        : "indonesian",
    "preference_name" : "my_lang_1"}';
    begin
    DBMS_VECTOR_CHAIN.CREATE_LANG_DATA(
    JSON (params));
    end;
    /
```
    

**End-to-end example**: 

To run an end-to-end example scenario using this procedure, see [Create and Use Custom Language Data](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-444A775F-AE3F-4F9E-8F50-67A273C9BCC2). 

**Related Topics**

  * [VECTOR_CHUNKS](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-5927E2FA-6419-4744-A7CB-3E62DBB027AD)
  * [UTL_TO_CHUNKS](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-4E145629-7098-4C7C-804F-FC85D1F24240)
  * [Text Processing Views](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-E2B9F02C-E2A6-439B-9A2E-177FF7FA6EE0)



**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
