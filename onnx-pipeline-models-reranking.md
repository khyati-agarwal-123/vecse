## ONNX Pipeline Models: Reranking Pipeline {#GUID-0A56CD32-B289-424F-B84F-1DD346F177B0}

ONNX pipeline models provide a reranking pipeline that calculates similarity score for a given pair of texts.

Reranking Pipeline

Reranking models (also known as cross-encoders or rerankers) calculate a similarity score for given pairs of texts. Instead of encoding the input text into a fixed-length vector as [text embedding](onnx-pipeline-models-text-embedding.md#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6) models do, rerankers encode two input texts simultaneously and produces a similarity score for the pair. By default it outputs a logit which can be converted to a number between 0 and 1 by applying a sigmoid activation function on it. Although you can compute similarity score between two texts through embedding models by first computing the embeddings of the texts and then applying cosine similarity, the re-ranker models usually provide superior performance and they can be used to re-rank the top-k results from an embedding model. 

  1. **Input**: There are two inputs to the reranking pipeline named `first_input` and `second_input`. Each contains an array of one or more text strings. The two inputs should have equal number of text strings. An array will be produced for each pair of texts from `first_input` and`second_input`. For example, for the following input pairs two arrays are produced, one for the pair`'hi'` and `'hi'` and the other for the pair `'halloween'` and `'hello'`: 

`{'first_input':['hi','halloween'],'second_input':['hi','hello']}`

  2. **Pre-Processing**: 

Pre-processing for reranking pipeline includes tokenization. The tokenizer class: `transformers.models.xlm_roberta.tokenization_xlm_roberta.XLMRobertaTokenizer` is supported in OML4Py 2.1 for reranking models. 

  3. **Output**: The output of a reranking pipeline is an array of similarity scores, one for each input text pair. For the above example in the Input section, the output similarity score is 9.290878 for the pair `'hi' `and` 'hi'` and 4.3913193 for the pair `'halloween'` and `'hello'`. 



Reranking Pipeline Examples

  1. Generating a Reranking Pipeline:
```
    mn = 'BAAI/bge-reranker-base'
    em = ONNXPipeline(mn, function=MiningFunction.REGRESSION.name)
    mf = 'bge-reranker-base'
    em.export2file(mf)
```
    

  2. Importing the reranker model to the database: 
```
    BEGIN
    DBMS_VECTOR.LOAD_ONNX_MODEL('DM_DUMP','bge-reranker-base.onnx','doc_model', JSON('{"function" : "regression",
    "regressionOutput" : "logits", "input":{"first_input": ["DATA1"],"second_input": ["DATA2"]}}'));
    END;
```
    

  3. Obtain the similarity score in the database:
```
    SELECT prediction(doc_model USING 'what is panda?' as DATA1, 'hi' as DATA2) from dual;
```
    

The above example produces a similarity score between two text strings using the re-ranker ONNX pipeline imported in step 2. The score range from -infinity to +infinity. Score close to +infinity indicates a high similarity between the two strings, and score closer to -infinity indicate low similarity between the strings. A sigmoid function can map the similarity score to a float value in the range [0,1]. A sample output score corresponding to the above example is shown here: 
```
    PREDICTION(DOC_MODELUSING'WHATISPANDA?'ASDATA1,'HI'ASDATA2)
    -----------------------------------------------------------
    -7.73E+000
```
    




**Parent topic:** [Import Pretrained Models in ONNX Format](import-pretrained-models-onnx-format-vector-generation-database.md)
