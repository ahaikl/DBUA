---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Use SQL Functions to Generate Embeddings

Choose to implement Vector Utility SQL functions to perform parallel or on-the-fly chunking and embedding operations, within Oracle Database.

Vector Utility SQL functions are intended for a direct and quick interaction with data, within pure SQL.

To get chunks, this function uses the in-house implementation with Oracle Database. To get an embedding, this function uses ONNX embedding models that you load into the database \(and not third-party REST providers\).

## VECTOR\_CHUNKS

Use the `VECTOR_CHUNKS` SQL function if you want to split plain text into chunks \(pieces of words, sentences, or paragraphs\) in preparation for the generation of embeddings, to be used with a vector index.

For example, you can use this function to build a standalone Text Chunking system that lets you break down a large amount of text into smaller yet semantically meaningful chunks. You can experiment with your chunks by inspecting and accordingly amending the chunking results and then proceed further.

For detailed information, see [VECTOR\_CHUNKS](olink:SQLRF-GUID-5927E2FA-6419-4744-A7CB-3E62DBB027AD) in *Oracle Database SQL Language Reference*.

## VECTOR\_EMBEDDING

Use the `VECTOR_EMBEDDING` function if you want to generate a single vector embedding for different data types.

For example, you can use this function in information-retrieval applications or chatbots, where you can generate a query vector on the fly from a user's natural language text input.

For detailed information, see [VECTOR\_EMBEDDING](olink:SQLRF-GUID-5ED78260-6D21-4B6B-86E0-A1E70EFA11CA) in *Oracle Database SQL Language Reference*.

**Parent topic:**[Generate Vector Embeddings Using Vector Utilities Leveraging Third-Party REST APIs](GUID-29B9E7E1-5A99-4D95-8614-58CA07D29957.md)

