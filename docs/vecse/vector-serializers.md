---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Vector Serializers

`FROM_VECTOR()` and `VECTOR_SERIALIZE()` are synonymous serializers of vectors.

Both functions take a vector as input and return a string of type `VARCHAR2` or `CLOB` as output. They optionally take a `RETURNING` clause to specify the returned data type. If `VARCHAR2` is specified without size, the returned value size is 32767. There is no support to convert to `CHAR`, `NCHAR`, and `NVARCHAR2`.

For detailed information, see the following in the *Oracle SQL Language Reference*:

-   [FROM\_VECTOR](olink:SQLRF-GUID-AA60B3CB-FCB7-4944-9E06-976C272855B1)

-   [VECTOR\_SERIALIZE](olink:SQLRF-GUID-9E3FFB34-F924-4C02-B35D-30B9FA1DA1A3)


**Parent topic:**[Other Basic Vector Functions](GUID-DC9AFA58-8D18-4419-9BF7-18A788B66EE4.md)

## FROM\_VECTOR

### Syntax

```
FROM_VECTOR (* expr *[ RETURNING ( CLOB | VARCHAR2 [ ( size [BYTE | CHAR] ) ] ) ] )
```

![](GUID-B3B0B5D2-A102-484B-98C5-D28F91E39107-default.gif)

## VECTOR\_SERIALIZE

### Syntax

```
VECTOR_SERIALIZE (* expr *[ RETURNING ( CLOB | VARCHAR2 [ ( size [BYTE | CHAR] ) ] ) ] )
```

![](GUID-73F1E87C-14F1-4FFB-9857-F3598ED292D5-default.gif)

## Parameters

`*expr*` must have a vector type. The function returns `NULL` if `*expr*` is `NULL`.

## Examples

```
SELECT FROM_VECTOR(TO_VECTOR('[1, 2, 3]') );

SELECT FROM_VECTOR(TO_VECTOR('[1.1, 2.2, 3.3]', 3, FLOAT32) );

VECTOR_SERIALIZE(VECTOR('[1.1,2.2,3.3]',3,FLOAT32))
---------------------------------------------------------------
[1.10000002E+000,2.20000005E+000,3.29999995E+000]

1 row selected.

SELECT FROM_VECTOR( TO_VECTOR('[1.1, 2.2, 3.3]', 3, FLOAT32) RETURNING VARCHAR2(1000));

VECTOR_SERIALIZE(VECTOR('[...]',3,FLOAT32)RETURNINGVARCHAR2(1000))
------------------------------------------------------------------
[1.10000002E+000,2.20000005E+000,3.29999995E+000]

1 row selected.

SELECT FROM_VECTOR(TO_VECTOR('[1.1, 2.2, 3.3]', 3, FLOAT32) RETURNING CLOB );

VECTOR_SERIALIZE(VECTOR('[...]',3,FLOAT32)RETURNINGCLOB)
--------------------------------------------------------
[1.10000002E+000,2.20000005E+000,3.29999995E+000]

1 row selected.
```

**Note:**

Applications using Oracle Client 23ai libraries or Thin mode drivers can fetch vector data directly, as shown in the following example:

```
SELECT dataVec FROM vecTab;
```

For applications using pre-Oracle Client 23ai libraries connected to Oracle Database 23ai, use the `FROM_VECTOR()` SQL function to fetch vector data, as shown by the following example:

```
SELECT FROM_VECTOR(dataVec) FROM vecTab;
```

