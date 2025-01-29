## Import Pretrained Models in ONNX Format {#GUID-D8140BF9-08E9-4B3F-9E28-E40A6FD181A4}

This topic describes about ONNX Pipeline Models for converting pretrained models to ONNX format. 

Open Neural Network Exchange or ONNX is an open standard format of machine learning models. Consider the following use cases: 

  * Using complex media input such as text or image for similarity search
  * Perform text classification
  * Perform reranking



You need vector embedding of the media input to perform all the tasks mentioned above. ONNX pipeline models allows you to convert text and/or image models into ONNX format if they are not already in ONNX format, import the ONNX format models into Oracle Database, and generate embeddings from your data within the database. The ONNX format models imported to the database could be used for tasks such as search and classification.

Pretrained models are models that are already trained on a media data (text, image, etc.) and saved to a storage format for future use. [Hugging Face](https://huggingface.co/) is the most popular platform that hosts pretrained models typically created with PyTorch. 

The Python package `oml.utils` contains three classes: `ONNXPipeline`, `ONNXPipelineConfig`, and `MiningFunction`. The package handles ONNX pipeline generation and export, while also incorporating the necessary pre- and post-processing steps. 

  * **ONNXPipeline** : Allows you to import a pretrained model (Your own pretrained model or one of the supported models from Hugging Face) 
  * **ONNXPipelineConfig** : Allows you to configure the attributes of pretrained model (Your own pretrained model or one of the supported models from Hugging Face) 
  * **MiningFunction**: Allows you to choose from one of the following mining options: 
    * `EMBEDDING` : Corresponds to text and image embedding 
    * `CLASSIFICATION` : Corresponds to text classification 
    * `REGRESSION` : Corresponds to re-ranking 



> **note:** WARNING:EmbeddingModel and EmbeddingModelConfig are deprecated. Instead, please use ONNXPipeline and ONNXPipelineConfig respectively. The details of the deprecated classes can be found in [Python Classes to Convert Pretrained Models to ONNX Models (Deprecated)](python-classes-convert-pretrained-models-onnx-models-deprecated.md#GUID-53F409DB-CC9A-406E-8697-0A824A415914). If a you choose to use a deprecated class, a warning message will be shown indicating that the classes will be removed in the future and advising the user to switch to the new class. 

The ONNX pipeline models are available for [text embedding](onnx-pipeline-models-text-embedding.md#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6), [image embedding](onnx-pipeline-models-image-embedding.md#GUID-285F1B43-B822-48E2-B29D-4A33C6975140), [multi-modal embedding](onnx-pipeline-models-multi-modal-embedding.md#GUID-C44EA9C1-2DE0-4E34-AE3D-AFC658DF62AC), [text classification](onnx-pipeline-models-text-classification.md#GUID-828BA3C8-A6F2-4A35-998C-F6C091CB6170) and [re-ranking](onnx-pipeline-models-reranking.md#GUID-0A56CD32-B289-424F-B84F-1DD346F177B0). 

  * [ONNX Pipeline Models : Text Embedding](onnx-pipeline-models-text-embedding.md)  
ONNX pipeline models provides text embedding models that accepts text as input and produces embeddings as output. The pipeline models also provide the necessary pre-processing and post-processing needed for the text. 
  * [ONNX Pipeline Models : Image Embedding](onnx-pipeline-models-image-embedding.md)  
ONNX pipeline models provides image embedding models that accepts image as an input and produces embeddings as output. The pipeline models also provide the necessary pre-processing. 
  * [ONNX Pipeline Models : CLIP Multi-Modal Embedding](onnx-pipeline-models-multi-modal-embedding.md)  
ONNX pipeline models provides multi-modal embedding models that accepts both image and text as an input and produces embeddings as output. The pipeline models also provide the necessary pre-processing needed. 
  * [ONNX Pipeline Models: Text Classification](onnx-pipeline-models-text-classification.md)  
ONNX pipeline models provides text classification models that accepts text strings as input and produces the probablity of the input string belonging to a specific label. The pipeline models also provide the necessary pre-processing and post-processing. 
  * [ONNX Pipeline Models: Reranking Pipeline](onnx-pipeline-models-reranking.md)  
ONNX pipeline models provide a reranking pipeline that calculates similarity score for a given pair of texts. 
  * [Convert Pretrained Models to ONNX Model: End-to-End Instructions for Text Embedding](convert-pretrained-models-onnx-model-end-end-instructions.md)  
This section provides end-to-end instructions from installing the OML4Py client to downloading a pretrained embedding model in ONNX-format using the Python utility package offered by Oracle. 
  * [Import ONNX Models into Oracle Database End-to-End Example](import-onnx-models-oracle-database-end-end-example.md)  
Learn to import a pretrained embedding model that is in ONNX format and generate vector embeddings. 



**Parent topic:** [Generate Vector Embeddings](generate-vector-embeddings-node.md)
