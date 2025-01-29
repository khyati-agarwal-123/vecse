## JSON Metadata Parameters for ONNX Models {#GUID-B6D784CE-0A52-43B5-A92C-231358416A4B}

When importing models using the `IMPORT_ONNX_MODEL` (`DBMS_DATA_MINING`), ` LOAD_ONNX_MODEL` (`DBMS_VECTOR`), or `LOAD_ONNX_MODEL_CLOUD` (`DBMS_VECTOR`) procedures, you supply metadata as JSON parameters. 

Parameters

Field | Value Type | Description  
---|---|---  
`function` | String |  Specify regression, classification, clustering, or embedding. This is a mandatory setting. <br>**NOTE**: <br>The only JSON parameter required when importing the model is the machine learning function.   
`input` | NA | Describes the model input mapping. See "Input" in Usage Notes.  
`regressionOutput` | String | The name of the regression model output that stores the regression results. The output is expected to be a tensor of supported shape of any supported regression output type. See "Output" in Usage Notes.  
`classificationProbOutput` | String | The name of the classification model output storing probabilities. The output is expected to be a tensor value of type float (width 32/64) of supported shape. See "Automatic normalization of output probabilities" in Usage Notes.  
`clusteringDistanceOutput` | String | The name of the clustering model output storing distances. The output is of type float (width 16/32/64) of supported shape.  
`clusteringProbOutput` | String | The name of the clustering model output storing probabilities. The output is of type float (width 16/32/64) of supported shape.  
`classificationLabelOutput` | String |  The name of the model output holding label information.<br>You have the following metadata parameters to specify the labels for classification: <br>* `labels`: <br>specify the labels directly in the JSON metadata <br>`classificationLabelOutput`: <br>specify the model output that provides labels 
<br>If you do not specify any value for this parameter or the function of the model is not classification, you will receive an error.<br>The user can specify to use labels from the model directly by setting `classificationLabelOutput` to the model output holding the label information. The tensor output holding the label information must be the same size as the number of classes and must be of integer or string type. If the tensor that holds the labels is of string type, the returned type of the `PREDICTION` operator is `VARCHAR2`. If the tensor that holds the labels is of integer type, the returned type of the `PREDICTION` operator is `NUMBER`.   
`normalizeProb` | String | Describes automatic normalization of output probabilities. See "Automatic normalization of output probabilities" in Usage Notes.  
`labels` | NA |  The labels used for classification. <br>If you want to use custom labels, specify the labels using the labels field in the JSON metadata. The field can be set to an array of length equal to the number of classes. The labels for the class i must be stored at index i of the label array. If an array of strings is used, the returned type of the `PREDICTION` operator is `VARCHAR2`. The size of the string labels specified by the user cannot exceed *4000* bytes. If an array of numbers is used, the returned type of the `PREDICTION` operator is `NUMBER`. <br>If you do not specify labels or `classificationLabelOutput`, classes are identified by integers in the range 1 to N where N is the number of classes. In this case, the returned type of the `PREDICTION` operator is `NUMBER`.   
`embeddingOutput` | String | The model output that holds the generated embeddings.  
`suitableDistanceMetrics` | String | An array of names of suitable distance metrics for the model. The names must be the names of the distance metrics used for the Oracle `VECTOR_DISTANCE` operator. To know the supported distance metrics, see [Vector Distance Metrics](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-DBC136C1-7C63-4B7F-902B-2289FF375560#GUID-DBC136C1-7C63-4B7F-902B-2289FF375560).   
`normalization` | Boolean | A boolean value indicates if normalization is applied to the output vector. The value 1 means normalization is applied. Normalization is process of converting an embedding vector so that it's norm or length equals 1. A normalized vector maintains its direction but its length becomes 1. The resulting vector is often called a unit vector.  
`maxSequenceLength` | Number | The maximum length of the token (input) sequence that is meaningful for the model. This parameter sets a limit on the number of tokens, words, or elements in each input sequence that the model will process. This ensures uniform input size for the model. For example, the value could be 128, or 512 to 4096 depending on the task for which the parameter is used. A machine translation model might have a `maxSequenceLength` of 512, accommodating sentences or paragraphs up to 512 tokens for translation tasks.   
`pooling` | String | Indicates the pooling function performed on the output vector.  
`modelDescription` | Object | A JSON object that allows users to add additional descriptions to the models complementing the existing ONNX metadata for model description.  
`languages` | String | A comma-separated list of language name or abbreviation, as described in *"A.1 Languages"* of *Oracle Database Globalization Support Guide*. If you import multi-lingual embedding model, specify the language or the language abbreviation as the metadata.   
`tokenizer` | String | Tokenizers help in transforming text into words. There are several tokenizers available, including: <br>bert, gpt2, bpe, wordpiece, sentencepiece, and clip.  
`embeddingLayer` | String | An identifier for the embedding layer. An embedding layer, serving as a hidden layer in neural networks, transforms input data from high to lower dimensions, enhancing the network's understanding of input relationships and data processing efficiency. Embedding layer helps in processing and analyzing categorical or discrete data. It achieves this by transforming categories into continuous embeddings, capturing the essential semantic relationships and similarities between them. For example the last hidden state in some transformer, or a layer in a resnet network.  
`defaultOnNull` | NA |  Specify the replacement of missing values in the JSON using the `defaultOnNull` field. If `defaultOnNull` is not specified, the replacement of missing values is not performed. The `defaultOnNull` sets the missing values to NULL by default. You can override the default value of NULL by providing meaningful default values to substitute for NULL. The field must be a JSON object literal, whose fields are the input attribute names and whose values are the default values for the input. Note that the default value is of type string and must be a valid Oracle PL/SQL NVL value for the given datatype.   
  
**Note:** The parameters are case-sensitive. A number of default conventions for output parameter names and default values allows to minimize the information that you may have to provide. The parameters such as `suitableDistanceMetrics` are informational only and you are not expected to provide this information while importing the model. The JSON descriptor may specify only one input attribute. If more are specified, you will receive an error. You will receive an error if the `normalizeProb` field is specified as the JSON metadata parameter. 

Usage Notes

The name of the model follows the same restrictions as those used for other machine learning models, namely:

  * **Input**

When importing a model from an ONNX representation, you must specify the name of the attribute used for scoring and how it maps to actual ONNX inputs. A scoring operator uses these attribute names to identify the columns to be used. (For example, `PREDICTION` ). Follow these conventions to specify the attribute names using the input field: 

not specified: When the field input is not specified, attribute names are mapped directly to model inputs by name. That is, if the attribute name is not specified in the JSON metadata, then the name of the input tensor is used as an attribute name. Each model input must have dimension `[batch_size, value]`. If you do not specify `input` in the JSON metatdata, the value must be 1. You don’t have to specify extra metadata if the input of the model already conforms to the format. For an embedding model, a single input is provided that may be used in batches. Here, if the `input` parameter is not specified in the JSON metadata, the valid model will have `[batch_size, 1]`. 

You must ensure that all attribute names, whether implied by the model or explicitly set by you through the input field, are valid Oracle Database identifiers for column names. Each attribute name within a model must be unique, ensuring no duplicates exist.

You can explicitly specify attribute name for model that use input tensors that have a dimension larger than 1 (for example, (batch_size, 2)). In this case, you must specify a name for each of these values for them to be interpreted as independent attribute name. This can be done for regression, classification, clustering which are models whose scoring operation can take multiple input attributes.

  * **Output**

As models might have multiple outputs, you can specify which output is of interest for a specific machine learning technique. You have the following ways to specify model outputs:

    * Specify the output name of interest in the JSON during model import. If the specified name is not a valid model output (see the table with valid outputs for a given machine learning function), you will receive an error.
    * If the model produces an output that matches the expected output name for the given machine learning technique (for example, `classificationProbOutput`) and you didn't explicitly specify it, the output is automatically assumed. 
    * If you do not specify any output name and the model has a single output, the system assumes that the single output corresponds to a default specific to the machine learning technique. For an embedding machine learning function, the default value is `embeddingOutput`. 

The system reports an error if you do not specify model outputs or if you supply outputs that the specified machine learning function does not support. The following table displays supported outputs for a specific machine learning function:

Machine learning function | Output  
---|---  
regression | `regressionOutput`  
classification | `classificationProbOutput`  
clustering | `clusteringDistanceOutput`  
embedding | `embeddingOutput`  
  
If none of the mentioned model outputs are specified, or if you supply outputs that are not supported by the specified machine learning function, you will receive an error. 

  * **Automatic Normalization of Output Probabilities**

Many users widely employ the softmax function to normalize the output of multi-class classification models, as it enables to easily interpret the results of these models. The **softmax function** is a mathematical function that converts a vector of real numbers into a probability distribution. It is also known as the softargmax, or normalized exponential function. This function is available to you to specify at the model import-time that a softmax normalization must be applied to the tensor holding output probabilities such as `classificationProbOutput` and `clusteringProbOutput`. Specify `normalizeProb` to define the normalization that must be applied for softmax normalization. The default setting is `none`, indicating that no normalization is applied. You can choose `softmax` to apply a softmax function to the probability output. Specifying any other value for this field will result in an error during import. Additionally, specifying this field for models other than classification and clustering will also lead to an error. 




Example: Specifying JSON Metadata Parameters for Embedding Models

The following example illustrates a simple case of how you can specify JSON metadata parameters while importing an ONNX embedding model into the Database using the `DBMS_VECTOR.IMPORT_ONNX_MODEL` procedure. 
```
    DBMS_VECTOR.IMPORT_ONNX_MODEL('my_embedding_model.onnx', 'doc_model',
    JSON('{"function" : "embedding",
    "embeddingOutput" : "embedding" ,
    "input":{"input": ["DATA"]}}'));
```
    

Example: Specifying Complete JSON Metadata Parameters for Embedding Models

The following example illustrates how to provide a complete JSON metadata parameters, with an exception of `embeddingLayer`, for importing embedding models. 
```
    DECLARE
    metadata JSON;
    mdtxt varchar2(4000);
    BEGIN
    metadata := JSON(q'#
    {
    "function"                : "embedding",
    "embeddingOutput"         : "embedding",
    "input"                   : { "input" : ["txt"]},
    "maxSequenceLength"       : 512,
    "tokenizer"               : "bert",
    "suitableDistanceMetrics" : [ "DOT", "COSINE", "EUCLIDEAN"],
    "pooling"                 : "Mean Pooling",
    "normalization"           : true,
    "languages"               : ["US"],
    "modelDescription"        : {
    "description" : "This model was tuned for semantic search: Given a query/question, if can find relevant passages. It was trained on a large and diverse set of (question, a
    nswer) pairs.",
    "url" : "https://example.co/sentence-transformers/my_embedding_model"}
    }
    #');
    -- load the onnx model
    DBMS_VECTOR.IMPORT_ONNX_MODEL('my_embedding_model.onnx', 'doc_model', metadata);
    END;
    /
```
    

> **note:** See Also: [ *Oracle Machine Learning for SQL User’s Guide*](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=DMPRG-GUID-A39EB438-4FAE-45C4-9E6B-FBEC7FA84BE8) for examples of using ONNX models for machine learning tasks 

**Parent topic:** [LOAD_ONNX_MODEL](load_onnx_model-procedure.md)
