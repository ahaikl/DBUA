---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Shorthand Operators for Distances

Oracle AI Vector Search provides shorthand operators that you can use for distance functions.

## Purpose

Oracle provides the following shorthand distance operators in lieu of their corresponding distance functions:

-   `<->` is the Euclidian distance operator: `expr1 <-> expr2` is equivalent to `L2_DISTANCE(expr1, expr2)` or `VECTOR_DISTANCE(expr1, expr2, EUCLIDEAN)`
-   `<=>` is the cosine distance operator: `expr1 <=> expr2` is equivalent to`COSINE_DISTANCE(expr1, expr2)` or `VECTOR_DISTANCE(expr1, expr2, COSINE)`
-   `<#>` is the negative dot product operator: `expr1 <#> expr2` is equivalent to `-1*INNER_PRODUCT(expr1, expr2)` or `VECTOR_DISTANCE(expr1, expr2, DOT)`

## Syntax

`<expr1> <-> <expr2>`

`<expr1> <#> <expr2>`

`<expr1> <=> <expr2>`

## Parameters

`expr1` and `expr2` must evaluate to vectors and have the same number of dimensions. These operations return `NULL` if either `expr1` or `expr2` is `NULL`.

## Example

-   `'[1, 2]' <-> '[0,1]'`
-   `v1 <-> '[' || '1,2,3' || ']'` is equivalent to `v1 <-> '[1, 2, 3]'`
-   `v1 <-> '[1,2]'` is equivalent to `L2_DISTANCE(v1, '[1,2]')`
-   `v1 <=> v2` is equivalent to `COSINE_DISTANCE(v1, v2)`
-   `v1 <#> v2` is equivalent to `-1*INNER_PRODUCT(v1, v2)`

**Parent topic:**[Vector Distance Functions](GUID-8667D062-8084-4E55-8BD7-2C84E05F52FA.md)

