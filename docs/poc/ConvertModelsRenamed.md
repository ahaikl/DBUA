---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Convert Pretrained Models to ONNX Format

OML4Py enables the use of text transformers from Hugging Face by converting them into ONNX format models. OML4Py also adds the necessary tokenization and post-processing. The resulting ONNX pipeline is then imported into the database and can be used to generate embeddings for AI Vector Search.

**Note:** This feature will **only** work on OML4Py client. It is not supported on the OML4Py server.

If you do not have a pretrained embedding model in ONNX-format to generate embeddings for your data, Oracle offers a Python utility package that downloads pretrained models from an external source, converts the model to ONNX format augmented with pre-processing and post-processing steps, and imports the resulting ONNX-format model into Oracle Database. Use the `DBMS_VECTOR.LOAD_ONNX_MODEL` procedure to import the file as a mining model. Then leverage the in-database ONNX Runtime with the ONNX model to produce vector embeddings.

At a high level, the Python utility package performs the following tasks:

-   Downloads the pretrained model from external source to your system
-   Augments the model with pre-processing and post-processing steps and creates a new ONNX model
-   Validates the augmented ONNX model
-   Loads into the database as a mining model or optionally exports to a file

The Python utility can take any of the models in the preconfigured list as input. Alternatively, you can use the built-in template that contains common configurations for certain groups of models such as text-based models. To understand what a preconfigured list, what is a built-in template is, and how to use them, read further.

## Limitations

This table describes the limitations of the Python utility package.

**Note:** This feature is available with the OML4Py client only.

|Parameter|Description|
|---------|-----------|
|`Transformer Model Type`|Currently supported only for text transformers.|
|`Model Size`|Model size should be less than 1GB. Quantization can help reduce the size.|
|`Tokenizers`|Must be either `BERT`, `GPT2`, `SENTENCEPIECE`, or `ROBERTA`.|

## Preconfigured List of Models

Preconfigured list of models are common models from external resource repositories that are provided with the Python utility. The preconfigured models have an existing specification. Users can create their own specification using the text template as a starting point. To get a list of all model names in the preconfigured list, you can use the `show_preconfigured` function.

## Templates

The Python utility package provides built-in text template for you to configure the pretrained models with pre-processing and post-processing operations. The template has a default specification for the pretrained models. This specification can be changed or augmented to create custom configurations. The text template uses Mean Pooling and Normalization as post-processing operations by default.

The Python utility package provides the following classes:

-   `EmbeddingModelConfig`

-   `EmbeddingModel`


To learn more about the Python classes, their properties, and to configure the properties, see [Python Classes to Convert Pretrained Models to ONNX Models](GUID-53F409DB-CC9A-406E-8697-0A824A415914.md#).

To use the Python utility, ensure that you have the following:

-   OML4Py Client running on Linux X64 for On-Premises Databases

-   Python 3.12 \(the earlier versions are not compatible\)


.

1.  Start Python in your work directory.

    ```
    $python3
    ```

    ```nocopybutton
    Python 3.12.2 | (main, Feb 27 2024, 17:35:02) [GCC 11.2.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    ```

2.  On the OML4Py client, load the Python classes:

    ```language-python
    from oml.utils import EmbeddingModel, EmbeddingModelConfig
    ```

3.  You can get a list of all preconfigured models by running the following:

    ```language-python
    EmbeddingModelConfig.show_preconfigured()
    ```

4.  To get a list of available templates:

    ```
    EmbeddingModelConfig.show_templates()
    ```

5.  Choose from:

    -   Generate an ONNX file from the preconfigureded model "sentence-transformers/all-MiniLM-L6-v2":

        ```language-python
        
        #generate from preconfigureded model "sentence-transformers/all-MiniLM-L6-v2"
        em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2")
        em.export2file("your_preconfig_file_name",output_dir=".")
        ```

    -   Generate an ONNX model from the preconfigured model "sentence-transformers/all-MiniLM-L6-v2" in the database:

        ```language-python
        
        #generate from preconfigureded model "sentence-transformers/all-MiniLM-L6-v2"
        em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2")
        em.export2db("your_preconfig_model_name")
        ```

    -   Generate an ONNX file using the provided text template:

        ```language-python
        #generate using the "text" template
        config = EmbeddingModelConfig.from_template("text",max_seq_length=512)
        em = EmbeddingModel(model_name="intfloat/e5-small-v2",config=config)
        em.export2file("your_template_file_name",output_dir=".")
        ```

    Let's understand the code:

    `from oml.utils import EmbeddingModel, EmbeddingModelConfig`: This line imports two classes, `EmbeddingModel` and `EmbeddingModelConfig`.

    In the preconfigured models first example:

    -   `em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2")` creates an instance of the `EmbeddingModel` class, loading a pretrained model specified by the `model_name` parameter. `em` is the embedding model object. `sentence-transformers/all-MiniLM-L6-v2` is the model name for computing sentence embeddings. This is the model name under Hugging Face. Oracle supports models from Hugging Face.

    -   The `export2file` command creates an ONNX format model with a user-specified model name in the database. `your_preconfig_file_name` is a user defined ONNX model file name.

    -   `output_dir="."` specifies the output directory where the file will be saved. The `"."` denotes the current directory \(that is, the directory from which the script is running\).

    In the preconfigured models second example:

    -   `em = EmbeddingModel(model_name="sentence-transformers/all-MiniLM-L6-v2")` creates an instance of the `EmbeddingModel` class, loading a pretrained model specified by the `model_name` parameter. `em` is the embedding model object. `sentence-transformers/all-MiniLM-L6-v2` is the model name for computing sentence embeddings. This is the model name under Hugging Face. Oracle supports models from Hugging Face.

    -   The `export2db` command creates an ONNX format model with a user defined model name in the database. `your_preconfig_model_name` is a user defined ONNX model name.
    In the template example:

    -   `config = EmbeddingModelConfig.from_template("text", max_seq_length=512)`: This line creates a configuration object for an embedding model using a method called `from_template`. The `"text"` argument indicates the name of the template. The `max_seq_length=512` parameter specifies the maximum length of input to the model as number of tokens. There is no default value. Specify this value for models that are not preconfigured.

    -   `em = EmbeddingModel(model_name="intfloat/e5-small-v2", config=config)` initializes an `EmbeddingModel` instance with a specific model and the previously defined configuration. The `model_name="intfloat/e5-small-v2"` argument specifies the name or identifier of the pretrained model to be loaded.

    -   The `export2file` command creates an ONNX format model with a user defined model name in the database. `your_template_file_name` is a user defined ONNX model name.

    -   `output_dir="."` specifies the output directory where the file will be saved. The `"."` denotes the current directory \(that is, the directory from which the script is running\).

    **Note:**

    -   The model size is limited to 1 gigabyte. For models larger than 400MB, Oracle recommends quantization.

        Quantization reduces the model size by converting model weights from high-precision representation to low-precision format. The quantization option converts the weights to INT8. The smaller model size enables you to cache the model in shared memory further improving the performance.

    -   The `.onnx` file is created with opset version 17 and ir version 8. For more information about these version numbers, see [https://onnxruntime.ai/docs/reference/compatibility.html\#onnx-opset-support](https://onnxruntime.ai/docs/reference/compatibility.html#onnx-opset-support).

6.  Exit Python.

    ```
    exit()
    ```

7.  Inspect if the converted models are present in your directory.

    **Note:** ONNX files are only created when `export2file` is used. If `export2db` is used, no ONNX files will be generated.

    ```
    ls-ltr *.onnx
    ```

    ```nocopybutton
    your_preconfig_file_name.onnx
    your_template_file_name.onnx
    ```

    The Python utility package validates the embedding text model before you can run them using ONNX Runtime. Oracle supports ONNX embedding models that conform to `string` as input and `float32 [<vector dimension>]` as output.

    If the input or output of the model doesn't conform to the description, you receive an error during the import.


`DBMS_VECTOR.LOAD_ONNX_MODEL` or `DBMS_DATA_MINING.IMPORT_ONNX_MODEL` are only needed if `export2file` was used instead of `export2db`. Use the resulting ONNX format model in the `DBMS_VECTOR.LOAD_ONNX_MODEL` procedure or in the `DBMS_DATA_MINING.IMPORT_ONNX_MODEL` procedure and generate vector embeddings using the `VECTOR_EMBEDDING` SQL operator.

**See Also:**

-   [*Oracle Database SQL Language Reference*](olink:SQLRF-GUID-5ED78260-6D21-4B6B-86E0-A1E70EFA11CA) for information about the `VECTOR_EMBEDDING` SQL function
-   [*Oracle Database PL/SQL Packages and Types Reference*](olink:ARPLS-GUID-BBAAE33A-412B-455A-8FE0-81BE31120300) for information about the `IMPORT_ONNX_MODEL` procedure
-   [*Oracle Database PL/SQL Packages and Types Reference*](olink:ARPLS-GUID-7F1D7992-D8F7-4AD9-9BF6-6EFFC1B0617A) for information about the `LOAD_ONNX_MODEL` procedure
-   [*Oracle Machine Learning for SQL Concepts*](olink:DMCON-GUID-6AEA7A0E-78E0-4083-A126-4516EB98175A) for more information about importing pretrained embedding models in ONNX format and generating vector embeddings
-   [https://onnx.ai/onnx/intro/](https://onnx.ai/onnx/intro/) for ONNX documentation

-   **[Python Classes to Convert Pretrained Models to ONNX Models](GUID-53F409DB-CC9A-406E-8697-0A824A415914.md)**  
Explore the functions and attributes of the `EmbeddingModelConfig` class and `EmbeddingModel` class within Python. These classes are designed to configure pretrained embedding models.

**Parent topic:**[Import Pretrained Models in ONNX Format for Vector Generation Within the Database](GUID-D8140BF9-08E9-4B3F-9E28-E40A6FD181A4.md)

