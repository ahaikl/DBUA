---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# USER\_VECTOR\_VOCAB

The `USER_VECTOR_VOCAB` view displays all custom token vocabularies created by the current user.

|Column Name

|Data Type

|Description

|
|-------------|-----------|-------------|
|`VOCAB_NAME`

|`VARCHAR2(128)`

|User-specified name of the vocabulary \(for example, `DOC_VOCAB`\)

|
|`FORMAT`

|`VARCHAR2(4)`

|Format of the vocabulary, such as `XLM`, `BERT`, or `GPT2`

|
|`CASED`

|`VARCHAR2(7)`

|Character-casing of the vocabulary, that is, vocabulary to be treated as cased or uncased

|

**Parent topic:**[Vector Utilities-Related Views](GUID-E2B9F02C-E2A6-439B-9A2E-177FF7FA6EE0.md)

