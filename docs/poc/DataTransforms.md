---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Understand the Stages of Data Transformations

Your input data may travel through different stages before turning into a vector.

For example, you may first transform textual data \(such as a PDF document\) into plain text, then break the resulting text into smaller pieces of text \(chunks\) to finally create vector embeddings on each chunk.

As shown in the following image, input data passes through a pipeline of optional stages from *Text* to *Chunks* to *Tokens* to *Vectors*, with *Vector Index* as the endpoint:

![](GUID-517074AC-5686-4081-8277-0F7FFDC2FAFF-print.eps)

## Prepare: Text and Chunks

This stage prepares your unstructured data to ensure that it is in a format that can be processed by vector embedding models.

To prepare large unstructured textual data \(for example, a PDF or Word document\), you first transform the data into plain text and then pass the resulting text through *Chunker*. The chunker then splits the text into smaller chunks using a process known as chunking. A single document may be split into many chunks, each transformed into a vector.

Chunks are pieces of words \(to capture specific words or word pieces\), sentences \(to capture a specific meaning\), or paragraphs \(to capture broader themes\). Later, you will learn about several chunking parameters and techniques to define so that each chunk contains relevant and meaningful context.

## Embed: Tokens and Vectors

You now pass the extracted chunks as input to *Model* \(a declared vector embedding model\) for generating vector embeddings on each chunk. This stage makes these chunks searchable.

The chunks are first passed through *Tokenizer* of the model to split into words or word pieces, known as tokens. The model then embeds each token into a vector representation.

Tokenizers used by embedding models frequently have limits on the size of the input text they can deal with, so chunking is needed to avoid loss of text when generating embeddings. If input text is larger than the maximum input limit imposed by the model, then the text gets truncated unless it is split up into appropriate-sized segments or chunks. Chunking is not needed for smaller documents, text strings, or summarized texts that meet this maximum input limit.

The chunker must select a text size that approximates the maximum input limit of your model. The actual number of tokens depends on the specified tokenizer for the model, which typically uses a vocabulary list of words, numbers, punctuations, and pieces of tokens.

A vocabulary contains a set of tokens that are collected during a statistical training process. Each tokenizer uses a different vocabulary format to process text into tokens, that is, the BERT multilingual model uses Word-Piece encoding and the GPT model uses Byte-Pair encoding.

For example, the BERT model tokenizes the following four words:

`Embedding usecase for chunking`

as the following eight tokens and also includes `##` \(number signs\) to indicate non-initial pieces of words:

`Em ##bedd ##ing use ##case for chunk ##ing`

Vocabulary files are included as part of a model's distribution. You can supply a vocabulary file \(recognized by your model's tokenizer\) to the chunker beforehand, so that it can correctly estimate the token count of your input data when chunking.

## Populate and Query: Vector Indexes

Finally, you store the semantic content of your input data as vectors in *Vector Index*. You can now implement combined similarity and relational queries to retrieve relevant results using the extracted vectors.

**Parent topic:**[Generate Vector Embeddings Using Vector Utilities Leveraging Third-Party REST APIs](GUID-29B9E7E1-5A99-4D95-8614-58CA07D29957.md)

