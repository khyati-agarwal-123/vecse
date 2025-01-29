## ONNX Pipeline Models: Text Classification {#GUID-828BA3C8-A6F2-4A35-998C-F6C091CB6170}

ONNX pipeline models provides text classification models that accepts text strings as input and produces the probablity of the input string belonging to a specific label. The pipeline models also provide the necessary pre-processing and post-processing.

In addition to text embedding models, Hugging Face repository also hosts transformer models that can be used for text classification. These models are typically fine-tuned embedding models on which a classification "head" was appended. The head changes the output of the model from a vector of embeddings to a list of labels and probabilities. For text based classification tasks such as sentiment analysis, you can generate a Text Classification Pipeline using OML4Py 2.1.

Text Classification Pipeline

  1. **Input**: Input to the text classification pipeline is provided in the form of a batch, or array, of 1 ore more text strings. Each text string provided in the input will correspond to an output list containing the labels and probabilities. 
  2. **Pre-Processing**: Similar to text embedding pipeline, the text classification pipeline also configures a tokenizer for tolenizing the text inputs. The following tokenizer classes are supported in OML4Py 2.1: 

**Table: Tokenizer Classes available for Text Classifiaction Pipeline**

Tokenizer Class | Tokenizer Type  
---|---  
`transformers.models.bert.BertTokenizer` | `BERT`  
`transformers.models.clip.CLIPTokenizer` | `CLIP`  
`transformers.models.distilbert.DistilBertTokenizer` | `BERT`  
`transformers.models.gpt2.GPT2Tokenizer` | `GPT2`  
`transformers.models.mpnet.MPNetTokenizer` | `BERT`  
`transformers.models.roberta.tokenization_roberta.RobertaTokenizer` | `ROBERTA`  
`transformers.models.xlm_roberta.XLMRobertaTokenizer` | `SENTENCEPIECE`  
  
> **note:** A tokenizer is automatically configured based on the tokenizer class configured for the model on Hugging Face. Tokenizer classes are provided by the transformer library. If the tokenizer class configured for the model in Hugging Face is not supported by OML4Py 2.1, an error will be raised. 

  3. **Original Model**: The original model must be a pre-trained PyTorch model in Hugging Face repository or from the local file system. Models on the local file system must match the Hugging Face format. 
  4. **Post-Processing**: The text classification pipeline provides a softmax function for post-processing by default. The softmax function will normalize the set of scores (logits) produced by the model into a probability distribution where each value is normalized to a range of [0,1]and all values sum to 1. You can choose to not include the softmax post-processing by providing an empty list of post-processors when generating the pipeline in OML4Py 2.1. 
  5. **Output**: The output of the classification pipeline is a list of probabilities for each input string. The length of the list is equal to the total number of classification targets or labels. 



The pipeline generator will attempt to use label data provided by the model's config in Hugging Face. This metadata is typically found in the config.json file in the label2id property. If this property is not found in a model's config.json then the pipeline generator will use the metadata that you provide or default to using the output index as the label. If you provide label metadata, it takes priority over the metadata provided in the config.json on the Hugging Face repositpry. 

> **note:** Label metadata does not become part of the ONNX model. It is only applied when exporting the model to the db with export2db. 

Text Classification Pipeline Examples

In order to generate ONNX pipeline models for image classification, `MiningFunction` class is used from the python utilities. The `MiningFunction` class can take one of the three values: `EMBEDDING`, `CLASSIFICATION`, and `REGRESSION`. You can choose one depending on the task which you would like to perform. The `MiningFunction` enum is defined as follows : 
```
    class MiningFunction(Enum) :
    EMBEDDING = 1
    CLASSIFCATION = 2
    REGRESSION = 3
```
    

Since you are working on text classification pipeline, you would choose `CLASSIFICATION` or `2` (Enum) for the class `MiningFunction`

> **note:** WARNING:EmbeddingModel and EmbeddingModelConfig are deprecated. Instead, please use ONNXPipeline and ONNXPipelineConfig respectively. The details of the deprecated classes can be found in [Python Classes to Convert Pretrained Models to ONNX Models (Deprecated)](python-classes-convert-pretrained-models-onnx-models-deprecated.md#GUID-53F409DB-CC9A-406E-8697-0A824A415914). If a you choose to use a deprecated class, a warning message will be shown indicating that the classes will be removed in the future and advising the user to switch to the new class. 

  1. Example for generating a text pipeline with a template:
```
    from oml.utils import ONNXPipeline,ONNXPipelineConfig,MiningFunction
    config = ONNXPipelineConfig.from_template("text",max_seq_length=512)
    pipeline = ONNXPipeline("SamLowe/roberta-base-go_emotions",config=config,**function=MiningFunction.CLASSIFICATION**)
    pipeline.export2file("emotions","testouput")
```
    

  2. Importing the text classification pipeline generated in the above example in the DB (SQL) 

> **note:** The following code assumes that you have created a directory in PL/SQL named 'ONNX_IMPORT' 
```
    BEGIN
    DBMS_VECTOR.LOAD_ONNX_MODEL(
    'ONNX_IMPORT',
    'emotions.onnx',
    'emotions',
    JSON('{"function":"classification","classificationProbOutput":"logits","input":{"input":["DATA"]},
    "labels":["admiration","amusement","anger","annoyance","approval","caring","confusion","curiosity",
    "desire","disappointment","disapproval","disgust","embarrassment","excitement","fear","gratitude",
    "grief","joy","love","nervousness","optimism","pride","realization","relief","remorse","sadness","surprise","neut"]}')
    );
    END;
```
    

In this example the label metadata is being applied when invoking the `DBMS_VECTOR.LOAD_ONNX_MODEL` function to import the ONNX pipeline into the database. Please recall that the "labels" property is part of the JSON argument to the function and not a part of the ONNX file. The labels must be provided separately (if not provided only the numerical indices of the output will be used). When exporting directly to the database from OML4Py using the `export2db` function, you can either rely on the default label metadata provided in the model's configuration on Hugging Face, or provide your own label metadata via a template argument. 

  3. Example for exporting a text classification pipeline with default label metadata:
```
    from oml.utils importONNXPipeline,ONNXPipelineConfig,MiningFunction
    import oml
    config = ONNXPipelineConfig.from_template("text",max_seq_length=512,labels=["admiration","amusement","anger","annoyance","approval","caring",
    "confusion","curiosity","desire","disappointment","disapproval","disgust","embarrassment","excitement",
    "fear","gratitude","grief","joy","love","nervousness","optimism","pride","realization","relief",
    "remorse","sadness","surprise","neut"])
    pipeline = ONNXPipeline("SamLowe/roberta-base-go_emotions",config=config,function=MiningFunction.CLASSIFICATION)
    oml.connect("pyquser","pyquser",dsn="pydsn")
    pipeline.export2db("emotions")
```
    

In this example the pipeline is exported with label metadata that you provided in the template. This example is a combination of the examples provided in the previous two steps.

  4. Example for scoring a text classification pipeline:
```
    select prediction(emotions using 'Today is a good day' as data),
    prediction_probability(emotions using 'Today is a good day'as data) from dual;
```
    




**Parent topic:** [Import Pretrained Models in ONNX Format](import-pretrained-models-onnx-format-vector-generation-database.md)
