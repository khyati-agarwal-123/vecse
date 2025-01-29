## APython Classes to Convert Pretrained Models to ONNX Models (Deprecated) {#GUID-53F409DB-CC9A-406E-8697-0A824A415914}

This topic provides the functions, attributes and example usage of the deprecated python classes `EmbeddingModelConfig` and `EmbeddingModel`. 

Examples for converting pretrained text models to ONNX format using EmbeddingModel, EmbeddingModelConfig

> **note:** This API is updated for 23.7. The version used for 23.6 and below used python packages EmbeddingModel and EmbeddingModelConfig. These packages are replaced with ONNXPipeline and ONNXPipelineConfig respectively. Oracle recommends that you use the latest version. If a you choose to use a deprecated class, a warning message will be shown indicating that the classes will be removed in the future and advising the user to switch to the new class. You can find the details of the updated python classes in [Import Pretrained Models in ONNX Format](import-pretrained-models-onnx-format-vector-generation-database.md#GUID-D8140BF9-08E9-4B3F-9E28-E40A6FD181A4)

  1. To load the python classes on the OML4Py client :
```
    from oml.utils import EmbeddingModel, EmbeddingModelConfig
```
    

  2. You can get a list of all preconfigured models by running the following:
```
    EmbeddingModelConfig.show_preconfigured()
```
    

  3. To get a list of available templates:
```
    EmbeddingModelConfig.show_templates()
```
    

  4. Generate an ONNX file from the preconfigured model "sentence-transformers/all-MiniLM-L6-v2":
```
    em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2")
    em.export2file("your_preconfig_file_name",output_dir=".")
```
    

  5. Generate an ONNX model from the preconfigured model "sentence-transformers/all-MiniLM-L6-v2" in the database:
```
    em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2")
    em.export2db("your_preconfig_model_name")
```
    

  6. Generate an ONNX file using the provided text template:
```
    config = EmbeddingModelConfig.from_template("text",max_seq_length=512)
    em = EmbeddingModel(model_name="intfloat/e5-small-v2",config=config)
    em.export2file("your_template_file_name",output_dir=".")
```
    




Functions and Attributes of EmbeddingModelConfig

The `EmbeddingModelConfig` class contains the properties required for the package to perform downloading, exporting, augmenting, validation, and storing of an ONNX model. The class provides access to configuration properties using the dot operator. As a convenience, well-known configurations are provided as templates. 

Parameters

This table describes the functions and properties of the `EmbeddingModelConfig` class. 

Functions | Parameter Type | Returns | Description  
---|---|---|---  
`from_template(name,**kwargs)` | 

  * `name` (`String`): <br>The name of the template <br>`**kwargs`: <br>template properties to override or add 

| Instance of `EmbeddingModelConfig` | A static function that creates an `EmbeddingModelConfig` object based on a predefined template given by the name parameter. You can use named arguments to override the template properties.   
`show_templates()` | NA | List of existing templates | A static function that returns a list of existing templates by name.  
`show_preconfigured()` | 

  * `include_properties` (`bool,optional`): <br>A flag indicating whether properties should be included in the results. Defaults to `False` so only names will be included by default.  <br>`model_name` (`str,optional`): <br>A model name to filter by when including properties. This argument will be ignored if `include_properties` is `False`. Otherwise only the properties of this model will be included in the results. 

| A list of preconfigured model names or properties. | Shows a list of preconfigured model names, or properties. By default, this function returns a list of names only. If the properties are required, pass the `include_properties` parameter as `True`. The returned list will contain a single dict where each key of the dict is the name of a preconfigured model and the value is the property set for that model. Finally, if only a single set of properties for a specific model is required, pass the name of the model in the `model_name` parameter (the `include_properties` parameter should also be `True`). This will return a list of a single dict with the properties for the specified model.   
  
Template Properties

The text template has configuration properties shown below:
```
    "do_lower_case": true,
    "post_processors":[{"name":"Pooling","type":"mean"},{"name":"Normalize"}]
```
    

> **note:** 

All other properties in the Properties table will take the default values. Any property without a default value must be provided when creating the `EmbeddingModelConfig` instance. 

Properties

This table shows all properties that can be configured. preconfigured models already have these properties set to specific values. Templates will use the default values unless a user overrides it when using the `from_template` function on `EmbeddingModelConfig`. 

Property | Description  
---|---  
`post_processors` | An array of post_processors that will be loaded after the model is loaded or initialized. The list of known and supported post_processors is provided later in this section. Templates may define a list of post_processors for the types of models they support. Otherwise, an empty array is the default.  
`max_seq_length` | This property is applicable for text-based models only. The maximum length of input to the model as number of tokens. There is no default value. Specify this value for models that are not preconfigured.  
`do_lower_case` | Specifies whether or not to lowercase the input when tokenizing. The default value is `True`.   
`quantize_model` | Perform quantization on the model. This could greatly reduce the size of the model as well as speed up the process. It may however result in different results for the embedding vector (against the original model) and possibly small reduction in accuracy. The default value is `False`.   
`distance_metrics` | An array of names of suitable distance metrics for the model. The names must be name of distance metrics used for Oracle vector distance operator. Only used when exporting the model to the database. Supported list is ["`EUCLIDEAN`","`COSINE`","`MANHATTAN`","`HAMMING`","`DOT`","`EUCLIDEAN_SQUARED`", "`JACCARD`"]. The default value is an empty array.   
`languages` | An array of language (Abbreviation) supported in the Database. Only used when exporting the model to the database. For a supported list of languages, see [Languages](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=NLSGP-GUID-D2FCFD55-EDC3-473F-9832-AAB564457830). The default value is an empty array.   
`use_float16` | Specifies whether or not to convert the exported onnx model to float16. The default value is `False`.   
  
Properties of post_processors

This table describes the built-in post_processors and their configuration parameters.

post_processor | Parameters | Description  
---|---|---  
`Pooling` | 

  * `name`: <br>`Pooling`.  <br>`type`: <br>Valid values should be `mean`(Default), `max`, `cls`

|  The Pooling post_processor summarizes the output of the transformer model into a fixed-length vector.  
`Normalize` | 

  * `name`: <br>Specify `Normalize`

|  The Normalize post_processor bounds the vector values to a range using L2 normalization.  
`Dense` | 

  * `name`: <br>Dense <br>`in_features`: <br>Input feature size <br>`out_features`: <br>Output feature size <br>`bias`: <br>Whether to learn an additive bias. The default value is `True`. <br>`activation_function`: <br>Activation function of the dense layer. Currently only supports `Tanh` as the activation function. 

|  Applies transformation to the incoming data.  
  
Example: Configure post_processors 

In this example, you override the post_processors in the sentence-transformers template with a `Max` Pooling post_processor followed by Normalization. 
```
    config = EmbeddingModelConfig.from_template("text")
    config.post_processors = [{"name":"Pooling","type":"max"},{"name":"Normalize"}]
```
    

Functions and Attributes of EmbeddingModel

Use the `EmbeddingModel` class to convert transformer models to the ONNX format with post_processing steps embedded into the final model. 

Parameters

This table describes the signature and properties of the `EmbeddingModel` class. 

Functions | Parameters | Description  
---|---|---  
`EmbeddingModel(model_name,configuration=None,settings={})` | 

  * `model_name`: <br>The name of the model to be used. For example, `medicalai/ClinicalBERT`<br>`configuration`: <br>An initialized `EmbeddingModelConfig` object. This parameter must be specified when using a template. If not specified, the model will be assumed to be a preconfigured model. <br>`settings`: <br>A dictionary of various settings that are global and control various operations such as logging levels and locations for files. 

| Creates a new instance of the `EmbeddingModel` class.   
  
Settings

The settings object is a dictionary passed to the `EmbeddingModel` class. It provides global properties for the `EmbeddingModel` class that are used for non-model-specific operations, such as logging. 

Property | Default Value | Description  
---|---|---  
`cache_dir` | `$HOME/.cache/OML` | The base directory used for downloads. Model files will be downloaded from the repository to directories relative to the ` cache_dir`. If the ` cache_dir` does not exist at time of execution, it will be created.   
`logging_level` | `ERROR` | The level for logging. Valid values are [‘DEBUG’, ‘INFO’, ‘WARNING’, ‘ERROR’, ‘CRITICAL’].  > **note: <br>** This log level is also applied globally to all python packages and is also mapped to the ONNXRuntime libraries.   
`force_download` | `False` | Forces download of model files instead of reloading from cache.  
`ignore_checksum_error` | `False` | Ignores any errors caused by mismatch in checksums when using preconfigured models.  
  
Functions

This table describes the function and properties of the `EmbeddingModel` class. 

Function | Parameters | Description  
---|---|---  
`export2file(export_name,output_dir=None)` | 

  * `export_name(string)`: <br>The name of the file. The file will be saved with the file extension `.onnx`<br>`output_dir(string)`: <br>An optional output directory. If not specified the file will be saved to the current directory 

| Exports the model to a file.  
`export2db(export_name)` | 

  * `export_name`(`string`): <br>The name that will be used for the mining model object. This name must be compliant with existing rules for object names in the database. 

| Exports the model to the database.  
  
Example: Preconfigured Model

This example illustrates the preconfigured embedding model that comes with the Python package. You can use this model without any additional configurations.
```
    "sentence-transformers/distiluse-base-multilingual-cased-v2": {
    "max_seq_length": 128,
    "do_lower_case": false,
    "post_processors":[{"name":"Pooling","type":"mean"},{"name":"Dense","in_features":768, "out_features":512, "bias":true, "activation_function":"Tanh"}],
    "quantize_model":true,
    "distance_metrics": ["COSINE"],
    "languages": ["ar", "bg", "ca", "cs", "dk", "d", "us", "el", "et", "fa", "sf", "f", "frc", "gu", "iw", "hi", "hr", "hu", "hy", "in", "i", "ja", "ko", "lt",
    "lv", "mk", "mr", "ms", "n", "nl", "pl", "pt", "ptb", "ro", "ru", "sk", "sl", "sq", "lsr", "s", "th", "tr", "uk", "ur", "vn", "zhs", "zht"]
    }
```
    
