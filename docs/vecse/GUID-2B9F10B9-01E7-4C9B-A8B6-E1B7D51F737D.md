---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Vector Constructors

`TO_VECTOR()` and `VECTOR()` are synonymous constructors of vectors. The functions take a string of type `VARCHAR2` or `CLOB` as input and return a vector as output.

`TO_VECTOR()` and `VECTOR()` are synonymous constructors of vectors. The functions take a string of type `VARCHAR2` or `CLOB` as input and return a vector as output.

For detailed information, see the following in the *Oracle SQL Language Reference*:

-   [TO\_VECTOR](olink:SQLRF-GUID-2CCAB607-A28B-43F7-A71D-9800C0B9A380)

-   [VECTOR](olink:SQLRF-GUID-8A63005B-5512-4D20-954C-7A9DA877FE4B)


**Parent topic:**[Other Basic Vector Functions](GUID-DC9AFA58-8D18-4419-9BF7-18A788B66EE4.md)

## TO\_VECTOR

### Syntax

```
TO_VECTOR(expr [ , number_of_dimensions [ , format ] ] )
```

![](GUID-E64C984C-9F07-469D-8EFB-639AE2634C67-default.gif)

## VECTOR

### Syntax

```
VECTOR ( expr [ , number_of_dimensions [ , format ] ] )
```

![](GUID-21D18BD7-55ED-4571-89FA-11B5B8DD8CD8-default.gif)

## Parameters

-   `*expr*` must evaluate to either \(a\) a string that represents a vector \(accepted input data types for the string are character types, JSON, `CLOB`, and `BLOB`\) or \(b\) a vector. The valid string representation of a vector is in the form of an array of non-null numbers enclosed with a bracket and separated by commas, such as `'[1, 3.4, -05.60, 3e+4]'`. If it evaluates to a vector \(the input's data type is `VECTOR`\), the function serves the purpose of converting it to the specified or default format. The expression can be `NULL`, in which case the result is `NULL` as well. If a `BLOB` is used as the input parameter, it must represent the vector binary bytes.
-   `*number\_of\_dimensions*` must be a numeric value that describes the number of dimensions of the vector to construct. The number of dimensions may also be specified as an asterisk `*`, in which case the dimension is determined by `*expr*`.
-   `*format*` must be one of the following tokens: `INT8`, `FLOAT32`, `FLOAT64`, or `*`. This is the target internal storage format of the vector. If `*` is used, the format will be `FLOAT32`. Note that this behavior is a bit different than declaring a vector column. If you declare a column of type `VECTOR(3, *)`, then all inserted vectors will NOT have their storage format modified \(stored as is\).

## Examples

```
SELECT TO_VECTOR('[34.6, 77.8]');
SELECT TO_VECTOR('[34.6, 77.8]', 2, FLOAT32);

SELECT TO_VECTOR('[34.6, 77.8]', 2, FLOAT32) FROM dual;

TO_VECTOR('[34.6,77.8]',2,FLOAT32)
---------------------------------------------------------
[3.45999985E+001,7.78000031E+001]

1 row selected.

SELECT TO_VECTOR('[34.6, 77.8, -89.34]', 3, FLOAT32);

TO_VECTOR('[34.6,77.8,-89.34]',3,FLOAT32)
-----------------------------------------------------------
[3.45999985E+001,7.78000031E+001,-8.93399963E+001]

1 row selected.
```

**Note:**

-   Applications using Oracle Client 23ai libraries or Thin mode drivers can insert vector data directly as a string or a `CLOB`. For example:

    ```
    INSERT INTO vecTab VALUES ('[1.1, 2.9, 3.14]');
    ```

-   For applications using pre-Oracle Client 23ai libraries connected to Oracle Database 23ai, use the `TO_VECTOR()` SQL function to insert vector data. For example:

    ```
    INSERT INTO vecTab VALUES(TO_VECTOR('[1.1, 2.9, 3.14]'));
    ```


