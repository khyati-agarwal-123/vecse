## ONNX Pipeline Models : CLIP Multi-Modal Embedding {#GUID-C44EA9C1-2DE0-4E34-AE3D-AFC658DF62AC}

ONNX pipeline models provides multi-modal embedding models that accepts both image and text as an input and produces embeddings as output. The pipeline models also provide the necessary pre-processing needed.

CLIP Multi-Modal Embedding Pipeline

CLIP models are multi-modal which allows you to perform embedding on both text and image input data. The main advantage of this model is to do image-text similarity. You can compare vectors related to text snippets and a given image, to understand which text can better describe given image. The pipeline generator will generate two ONNX pipeline models for a pretrained CLIP model, distinguished by their suffixes. The pipeline model for images is suffixed with **_img** and the model for text is suffixed with **_txt**. The same models with their suffixes will be loaded to the database when using *export2db*. For performing CLIP related tasks such as image-text similarity, both models will need to be used at inference time. 

  1. **Input**: CLIP models consist of two pipelines: an image embedding pipeline and a text embedding pipeline. The Image pipeline takes images as described in the Image Embedding Pipeline section of [ONNX Pipeline Models : Image Embedding](onnx-pipeline-models-image-embedding.md#GUID-285F1B43-B822-48E2-B29D-4A33C6975140), and the text Pipeline takes text as described in [ONNX Pipeline Models : Text Embedding](onnx-pipeline-models-text-embedding.md#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6) (Text Embedding support was introduced in OML4Py 2.0). 
  2. **Pre-processing**: The image pipeline for CLIP models utilizes the same pre-processing strategy as described in the Image Embedding Pipeline section of [ONNX Pipeline Models : Image Embedding](onnx-pipeline-models-image-embedding.md#GUID-285F1B43-B822-48E2-B29D-4A33C6975140). That is, an image processor that matches the model's configuration is utilized to prepare the images. The text pipeline utilizes a specific tokenizer: the **CLIPTokenizer (transformers.models.clip.CLIPTokenizer)**. 
  3. **Post-processing**: As a post-processing step, Normalization is added to both the image and text pipelines. 
  4. **Output**: Both models will produce vectors of the same shape that can then be compared using some similarity measure. 



CLIP Multi-Modal Embedding Examples

  1. Exporting a pre-configured image model to a file: 

The following example will produce two pipelines called **clip_img.onnx** and **clip_txt.onnx** , which can be used for image and text embeddings respectively. 
```
    from oml.utils import ONNXPipeline
    pipeline = ONNXPipeline("openai/clip-vit-base-patch32")
    oml.export2file("clip")
```
    

  2. Exporting a pre-configured image model to the database: 

This example will produce two in-database models called **clip_img** and **clip_txt** , which can be used for image and text embeddings respectively. 
```
    fromoml.utils import ONNXPipeline
    import oml
    pipeline = ONNXPipeline("openai/clip-vit-base-patch32")
    oml.connect("pyquser","pyquser",dsn="pydsn")
    oml.export2db("clip")
```
    

  3. Exporting a non pre-configured with a template to a file: 

This example will work for clip models that are not preconfigured. It will create two files called **clip_14_img.onnx** and **clip_14_txt.onnx**.
```
    from oml.utils import ONNXPipeline, ONNXPipelineConfig
    config = ONNXPipelineConfig.from_template("multimodal_clip")
    pipeline = ONNXPipeline("openai/clip-vit-large-patch14",config=config)
    oml.export2file("clip_14")
```
    




**Parent topic:** [Import Pretrained Models in ONNX Format](import-pretrained-models-onnx-format-vector-generation-database.md)
