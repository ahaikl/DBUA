---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Use PL/SQL Packages to Generate Embeddings

Choose to implement Vector Utility PL/SQL packages to perform chunking, embedding, and text generation operations along with text processing and similarity search, within and outside Oracle Database.

Vector Utility PL/SQL APIs work with both the ONNX embedding models \(loaded into the database\) and third-party REST providers, such as Cohere, Google AI, Hugging Face, Oracle Cloud Infrastructure \(OCI\) Generative AI, OpenAI, or Vertex AI. 

These packages are made up of subprograms, such as *chainable utility functions* and *vector helper procedures*.

-   **[Terms of Using Vector Utility PL/SQL Packages](GUID-67609313-2A2F-4F08-8A0A-51613B6400FB.md)**  
You must understand the terms of using REST APIs that are part of Vector Utility PL/SQL packages.
-   **[About Chainable Utility Functions and Common Use Cases](GUID-63773EF5-071E-4C02-9EC9-D4E0BA2CE2A2.md)**  
These are intended to be a set of chainable and flexible "stages" through which you pass your input data to transform into a different representation, including vectors.
-   **[About Vector Helper Procedures](GUID-E97EE03F-C922-4C2D-82A7-38152CBF70EB.md)**  
Vector helper procedures let you configure authentication credentials and language-specific data, for use in chainable utility functions.
-   **[Supplied Vector Utility PL/SQL Packages](GUID-D73320C0-C2E3-4B67-A7C9-C0490DC9DE6C.md)**  
Use either a lightweight `DBMS_VECTOR` package or a more advanced `DBMS_VECTOR_CHAIN` package with full capabilities.
-   **[Supported Third-Party Provider Operations](GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09.md)**  
Review the list of third-party REST providers that are supported with Vector Utility PL/SQL packages and the corresponding API calls allowed for each of those.
-   **[Validate JSON Input Parameters](GUID-BD911C3C-D17F-4A8D-90BF-5A82A604D7C8.md)**  
You can optionally validate the structure of your JSON input to the `DBMS_VECTOR.UTL` and `DBMS_VECTOR_CHAIN.UTL` functions, which use JSON to define their input parameters.

**Parent topic:**[Generate Vector Embeddings Using Vector Utilities Leveraging Third-Party REST APIs](GUID-29B9E7E1-5A99-4D95-8614-58CA07D29957.md)

