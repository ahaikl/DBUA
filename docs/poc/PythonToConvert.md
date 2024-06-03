---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Python Classes to Convert Pretrained Models to ONNX Models

Explore the functions and attributes of the `EmbeddingModelConfig` class and `EmbeddingModel` class within Python. These classes are designed to configure pretrained embedding models.

## EmbeddingModelConfig

The `EmbeddingModelConfig` class contains the properties required for the package to perform downloading, exporting, augmenting, validation, and storing of an ONNX model. The class provides access to configuration properties using the dot operator. As a convenience, well-known configurations are provided as templates.

## Parameters

This table describes the functions and properties of the `EmbeddingModelConfig` class.

|Functions|Parameter Type|Returns|Description|
|---------|--------------|-------|-----------|
|`from_template(name,**kwargs)`|-   `name` \(`String`\): The name of the template
-   `**kwargs`: template properties to override or add

|Instance of `EmbeddingModelConfig`|A static function that creates an `EmbeddingModelConfig` object based on a predefined template given by the name parameter. You can use named arguments to override the template properties.|
|`show_templates()`|NA|List of existing templates|A static function that returns a list of existing templates by name.|
|`show_preconfigured()`|-   `include_properties` \(`bool,optional`\): A flag indicating whether properties should be included in the results. Defaults to `False` so only names will be included by default.

-   `model_name` \(`str,optional`\): A model name to filter by when including properties. This argument will be ignored if `include_properties` is `False`. Otherwise only the properties of this model will be included in the results.

|A list of preconfigured model names or properties.|Shows a list of preconfigured model names, or properties. By default, this function returns a list of names only. If the properties are required, pass the `include_properties` parameter as `True`. The returned list will contain a single dict where each key of the dict is the name of a preconfigured model and the value is the property set for that model. Finally, if only a single set of properties for a specific model is required, pass the name of the model in the `model_name` parameter \(the `include_properties` parameter should also be `True`\). This will return a list of a single dict with the properties for the specified model.|

## Template Properties

The text template has configuration properties shown below:

```
"do_lower_case": true,
"post_processors":[{"name":"Pooling","type":"mean"},{"name":"Normalize"}]
```

**Note:**

All other properties in the Properties table will take the default values. Any property without a default value must be provided when creating the `EmbeddingModelConfig` instance.

## Properties

This table shows all properties that can be configured. preconfigured models already have these properties set to specific values. Templates will use the default values unless a user overrides it when using the `from_template` function on `EmbeddingModelConfig`.

|Property|Description|
|--------|-----------|
|`post_processors`

|An array of post\_processors that will be loaded after the model is loaded or initialized. The list of known and supported post\_processors is provided later in this section. Templates may define a list of post\_processors for the types of models they support. Otherwise, an empty array is the default.|
|`max_seq_length`

|This property is applicable for text-based models only. The maximum length of input to the model as number of tokens. There is no default value. Specify this value for models that are not preconfigured.|
|`do_lower_case`

|Specifies whether or not to lowercase the input when tokenizing. The default value is `True`.|
|`quantize_model`

|Perform quantization on the model. This could greatly reduce the size of the model as well as speed up the process. It may however result in different results for the embedding vector \(against the original model\) and possibly small reduction in accuracy. The default value is `False`.|
|`distance_metrics`|An array of names of suitable distance metrics for the model. The names must be name of distance metrics used for Oracle vector distance operator. Only used when exporting the model to the database. Supported list is \["`EUCLIDEAN`","`COSINE`","`MANHATTAN`","`HAMMING`","`DOT`","`EUCLIDEAN_SQUARED`"\]. The default value is an empty array.|
|`languages`|A array of language \(Abbreviation\) supported in the Database. Only used when exporting the model to the database. For a supported list of languages, see [Languages](olink:NLSGP-GUID-D2FCFD55-EDC3-473F-9832-AAB564457830). The default value is an empty array.|
|`use_float16`|Specifies whether or not to convert the exported onnx model to float16. The default value is `False`.|

## Properties of post\_processors

This table describes the built-in post\_processors and their configuration parameters.

|post\_processor|Parameters|Description|
|---------------|----------|-----------|
|`Pooling`

|-   `name`: `Pooling`.

-   `type`: Valid values should be `mean`\(Default\), `max`, `cls`


|The Pooling post\_processor summarizes the output of the transformer model into a fixed-length vector.

|
|`Normalize`

|-   `name`: Specify `Normalize`


|The Normalize post\_processor bounds the vector values to a range using L2 normalization.

|
|`Dense`

|-   `name`: Dense
-   `in_features`: Input feature size
-   `out_features`: Output feature size
-   `bias`: Whether to learn an additive bias. The default value is `True`.
-   `activation_function`: Activation function of the dense layer. Currently only supports `Tanh` as the activation function.

|Applies transformation to the incoming data.

|

## Example: Configure post\_processors

In this example, you override the post\_processors in the sentence-transformers template with a `Max` Pooling post\_processor followed by Normalization.

```
config = EmbeddingModelConfig.from_template("text")
config.post_processors = [{"name":"Pooling","type":"max"},{"name":"Normalize"}]
```

## EmbeddingModel

Use the `EmbeddingModel` class to convert transformer models to the ONNX format with post\_processing steps embedded into the final model.

## Parameters

This table describes the signature and properties of the `EmbeddingModel` class.

|Functions|Parameters|Description|
|---------|----------|-----------|
|`EmbeddingModel(model_name,configuration=None,settings={})`|-   `model_name`: The name of the model to be used. For example, `medicalai/ClinicalBERT`
-   `configuration`: An initialized `EmbeddingModelConfig` object. This parameter must be specified when using a template. If not specified, the model will be assumed to be a preconfigured model.
-   `settings`: A dictionary of various settings that are global and control various operations such as logging levels and locations for files.

|Creates a new instance of the `EmbeddingModel` class.|

## Settings

The settings object is a dictionary passed to the `EmbeddingModel` class. It provides global properties for the `EmbeddingModel` class that are used for non-model-specific operations, such as logging.

|Property|Default Value|Description|
|--------|-------------|-----------|
|`cache_dir`|`$HOME/.cache/OML`|The base directory used for downloads. Model files will be downloaded from the repository to directories relative to the `cache_dir`. If the `cache_dir` does not exist at time of execution, it will be created.|
|`logging_level`|`ERROR`|The level for logging. Valid values are \[‘DEBUG’, ‘INFO’, ‘WARNING’, ‘ERROR’, ‘CRITICAL’\]. **Note:** This log level is also applied globally to all python packages and is also mapped to the ONNXRuntime libraries.

|
|`force_download`|`False`|Forces download of model files instead of reloading from cache.|
|`ignore_checksum_error`|`False`|Ignores any errors caused by mismatch in checksums when using preconfigured models.|

## Functions

This table describes the function and properties of the `EmbeddingModel` class.

|Function|Parameters|Description|
|--------|----------|-----------|
|`export2file(export_name,output_dir=None)`|-   `export_name(string)`: The name of the file. The file will be saved with the file extension `.onnx`
-   `output_dir(string)`: An optional output directory. If not specified the file will be saved to the current directory

|Exports the model to a file.|
|`export2db(export_name)`|-   `export_name`\(`string`\): The name that will be used for the mining model object. This name must be compliant with existing rules for object names in the database.

|Exports the model to the database.|

## Example: Preconfigured Model

This example illustrates the preconfigured embedding model that comes with the Python package. You can use this model without any additional configurations.

```nocopybutton
"sentence-transformers/distiluse-base-multilingual-cased-v2": {
        "max_seq_length": 128,
        "do_lower_case": false,
        "post_processors":[{"name":"Pooling","type":"mean"},{"name":"Dense","in_features":768, "out_features":512, "bias":true, "activation_function":"Tanh"}],
        "quantize_model":true,
        "distance_metrics": ["COSINE"],
        "languages": ["ar", "bg", "ca", "cs", "dk", "d", "us", "el", "et", "fa", "sf", "f", "frc", "gu", "iw", "hi", "hr", "hu", "hy", "in", "i", "ja", "ko", "lt", "lv", "mk", "mr", "ms", "n", "nl", "pl", "pt", "ptb", "ro", "ru", "sk", "sl", "sq", "lsr", "s", "th", "tr", "uk", "ur", "vn", "zhs", "zht"]
    }
```

**Parent topic:**[Convert Pretrained Models to ONNX Format](GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6.md)

