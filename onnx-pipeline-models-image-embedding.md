## ONNX Pipeline Models : Image Embedding {#GUID-285F1B43-B822-48E2-B29D-4A33C6975140}

ONNX pipeline models provides image embedding models that accepts image as an input and produces embeddings as output. The pipeline models also provide the necessary pre-processing. 

Image Embedding Pipeline

  1. **Input** : Input to the pipeline is an image. Each image is represented as an array of bytes. This can be a BLOB when using SQL or a similar in-memory byte representation like `io.ByetIO` in Python. 
  2. **Pre-Processing** : The image embedding pipeline utilizes an Image Processor in the pre-processing step. The Image Processor is dependent on the model configuration in the Hugging Face repository. When you provide a pretrained image model (e.g. "[google/vit-base-patch16-224](https://huggingface.co/google/vit-base-patch16-224)"), the pipeline builder will look up the Image Processor class from the model's configuration and determine if it's supported by looking up the Image Processor name in pipeline builder templates. If a match is found then the specific Image Processor is loaded and configured according to the template. The following table shows the relationship between pipeline builder templates and their corresponding Image Processor classes. Image Processor classes are provided by the transformer library. 

> **note:** You do not need to reference these templates when using preconfigured image models. However any non-preconfigured model is template driven and these templates can be used for such models. 

**Template** | **Image Processor Class**  
---|---  
Image_ViT |  transformers.ViTImageProcessor  
Image_ConvNext |  transformers.ConvNextImageProcessor  
  
Each Image Processor has several possible operations that will modify the input image in a specific way. Each of these operations is represented by a configuration in the Image Processor template. The modification of these operations and their respective configuration is not supported. You are able to view the configurations and should be aware of them. The following table shows the image operations and their respective configurations: 

**Table: Image Processor Configurations**

Image Processor Operation | Description  
---|---  
Decode |  Converts the compressed image to 3-D raster format.  
Resize |  Resizes the given image to the new shape.  
Rescale |  Rescales (element wise multiplication) an image with a numeric value.  
Normalize |  Normalizes (adjusts the intensity values of pixels to a desired range, often between 0 to 1) an image using the given mean and standard deviation (std) using the following formulation: <br>*(image - mean) / std*  
CenterCrop |  Crops the image with a fixed size from the center.  
  
> **note:** The Image pre-processors are considered to be fixed and modifications are not allowed. We are exposing these details for your information and understanding. 

  3. **Original Model**: Pre-trained pytorch model from Hugging Face repository. The repository also contain preconfigured models which have an existing specification. To get a list of all model names in the preconfigured list, you can use the `show_preconfigured` function. You can also create your own specification using the above mentioned templates as a starting point. 
  4. **Output** : The pipeline generates embeddings for each input image. 



Image Embedding Examples

  1. Exporting a pre-configured image model:
```
    from oml.utils import ONNXPipeline
    pipeline =ONNXPipeline("google/vit-base-patch16-224")
    pipeline.export2file("your_preconfig_model_name")
```
    

This example uses the `google/vit-base-patch16-224` model, which is a pre-configured model shipped in OML4Py 2.1. The pipeline is exported to a file which will result in a file**your_preconfig_model_name.onnx** in the current directory. This model can optionally be exported directly to the database using the pipeline.export2db function. Although the exported pipeline will just produce embeddings, not a classification output, it is possible to train an OML classification model to take these embeddings and produce the desired classification. 

  2. Exporting a non pre-configured model with a template: 

> **note:** Refer to this section for understanding the templates 
```
    from oml.utils import ONNXPipeline,ONNXPipelineConfig
    import oml
    config = ONNXPipelineConfig.from_template("Image_ViT")
    pipeline = ONNXPipeline("nateraw/food",config=config)
    oml.connect("pyquser","pyquser",dsn="pydsn")
    oml.export2db("your_preconfig_model_name")
```
    

In this example, since the `nateraw/food` model is not included as a preconfigured model in OML4Py 2.1, a template approach was chosen. Since this is a ViT based model, the ViT template is selected. 




The process of loading a image model converted to a ONNX format is very similar to the ONNX format model generated from text. Refer to steps 6-9 from [ONNX Pipeline Models : Text Embedding](onnx-pipeline-models-text-embedding.md#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6__CHOICES_P22_NSB_DCC) for understanding how to load a ONNX model and use it further. 

**Parent topic:** [Import Pretrained Models in ONNX Format](import-pretrained-models-onnx-format-vector-generation-database.md)
