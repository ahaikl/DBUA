---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Create Tables Using the VECTOR Data Type

You can declare a table's column as a `VECTOR` data type.

The following command shows a simple example:

```
CREATE TABLE my_vectors (id NUMBER, embedding VECTOR);
```

In this example, you don't have to specify the number of dimensions or their format, which are both optional. If you don't specify any of them, you can enter vectors of different dimensions with different formats. This is a simplification to help you get started with using vectors in Oracle Database.

**Note:** Such vectors typically arise from different embedding models. Vectors from different models \(providing a different semantic landscape\) are not comparable for use in similarity search.

Here's a more complex example that imposes more constraints on what you can store:

```
CREATE TABLE my_vectors (id NUMBER, embedding VECTOR(768, INT8)) ;
```

In this example, each vector that is stored:

-   Must have 768 dimensions, and
-   Each dimension will be formatted as an `INT8`.

The number of dimensions must be strictly greater than zero with no practical upper limit.

The possible dimension formats are:

-   `INT8` \(8-bit integers\)
-   `FLOAT32` \(32-bit IEEE floating-point numbers\)
-   `FLOAT64` \(64-bit IEEE floating-point numbers\)

Oracle Database automatically casts the values as needed.

The following table guides you through the possible declaration format for a `VECTOR` data type:

|Possible Declaration Format

|Explanation

|
|-----------------------------|-------------|
|`VECTOR`

|Vectors can have an arbitrary number of dimensions and formats.

|
|`VECTOR(*, *)`

|Vectors can have an arbitrary number of dimensions and formats. `VECTOR` and `VECTOR(*,*)` are equivalent.

|
|`VECTOR`\(number\_of\_dimensions, \*\)

|Vectors must all have the specified number of dimensions or an error is thrown. Every vector will have its dimensions stored without format modification.

|
|`VECTOR`\(number\_of\_dimensions\)

|Vectors must all have the specified number of dimensions or an error is thrown. Every vector will have its dimensions stored without format modification. `VECTOR`\(number\_of\_dimensions, \*\) and `VECTOR`\(number\_of\_dimensions\) are equivalent.

|
|`VECTOR`\(\*, dimension\_element\_format\)

|Vectors can have an arbitrary number of dimensions, but their format will be up-converted or down-converted to the specified dimension\_element\_format \(`INT8`, `FLOAT32`, or `FLOAT64`\).

|

A vector can be `NULL` but its dimensions cannot \(for example, you cannot have a `VECTOR` with a `NULL` dimension such as `[1.1, NULL, 2.2]`\).

The following SQL\*Plus code example shows how the system interprets various vector definitions:

```
CREATE TABLE my_vect_tab (
     v1 VECTOR(3, FLOAT32),
     v2 VECTOR(2, FLOAT64),
     v3 VECTOR(1, INT8),
     v4 VECTOR(1, *),
     v5 VECTOR(*, FLOAT32),
     v6 VECTOR(*, *),
     v7 VECTOR
   );

Table created.

DESC my_vect_tab;
 Name                        Null?    Type
 --------------------------- -------- ----------------------------
 V1                                   VECTOR(3 , FLOAT32)
 V2                                   VECTOR(2 , FLOAT64)
 V3                                   VECTOR(1 , INT8)
 V4                                   VECTOR(1 , *)
 V5                                   VECTOR(* , FLOAT32)
 V6                                   VECTOR(* , *)
 V7                                   VECTOR(* , *)
```

You currently cannot define `VECTOR` columns in/as:

-   External Tables
-   IOTs \(neither as Primary Key nor as non-Key column\)
-   Clusters/Cluster Tables
-   Global Temp Tables
-   \(Sub\)Partitioning Key
-   Primary Key
-   Foreign Key
-   Unique Constraint
-   Check Constraint
-   Default Value
-   Modify Column
-   MSSM tablespace \(only SYS user can create VECTORs as Basicfiles in MSSM tablespace\)
-   CQN queries
-   Non-vector indexes such as B-tree, Bitmap, Reverse Key, Text, Spatial indexes, etc

Oracle Database does not support the following SQL constructs with VECTOR columns:

-   Distinct, Count Distinct
-   Order By, Group By
-   Join condition
-   Comparison operators \(e.g. \>, <, =\) etc

**Parent topic:**[Store Vector Embeddings](GUID-DF5E5492-2E9E-4C38-838D-56567BEC17C1.md)

