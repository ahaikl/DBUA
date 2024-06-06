---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# SQL Quick Start

A set of SQL commands is provided to run a particular scenario that will help you understand Oracle AI Vector Search capabilities.

This quick start scenario introduces you to the `VECTOR` data type, which represents the semantic meaning behind your unstructured data. You will also use Vector Search SQL operators, allowing you to perform a similarity search to find vectors \(and thereby content\) that are similar to each other. Vector indexes are also created to help you accelerate similarity searches in an approximate manner. See [Overview of Oracle AI Vector Search](GUID-746EAA47-9ADA-4A77-82BB-64E8EF5309BE.md#) for more introductory information if needed.

The script chunks two Oracle Database Documentation books, assigns them corresponding vector embeddings, and shows you some similarity searches using vector indexes.

To run this script you need three files similar to the following:

-   my\_embedding\_model.onnx, which is an ONNX export of the corresponding embedding model. To create such a file, see [Convert Pretrained Models to ONNX Format](olink:VECSE-GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6#GUID-E7C08BA2-B2B9-4081-9050-B9EB3EA46FA6).
-   database-concepts23ai.pdf, which is the PDF file for [Oracle Database 23ai Oracle Database Concepts manual](https://docs.oracle.com/en/database/oracle/oracle-database/23/cncpt/database-concepts.pdf).
-   oracle-ai-vector-search-users-guide.pdf, which is the PDF file for this guide that you are reading.

**Note:** You can use other PDF files instead of the ones listed here. If you prefer, you can use another model of your choice as long as you can generate it as an .onnx file.

Let's start.

1.  Copy the files to your local directory.

    There is no script and you can use scp. For example, to copy the three files to a directory on your server. You can call that directory /my\_local\_dir.

    ```
    scp my_embedding_model.onnx /my_local_dir
    scp database-concepts23ai.pdf /my_local_dir
    scp oracle-ai-vector-search-users-guide.pdf /my_local_dir
    ```

2.  Create storage, user, and privileges.

    Here you create a new tablespace and a new user. You grant that user the `DB_DEVELOPER_ROLE` and create an Oracle directory to point to the PDF files. You grant the new user the possibility to read and write from/to that directory.

    ```
    sqlplus / as sysdba
    
    CREATE TABLESPACE tbs1
    DATAFILE 'tbs5.dbf' SIZE 20G AUTOEXTEND ON
    EXTENT MANAGEMENT LOCAL
    SEGMENT SPACE MANAGEMENT AUTO;
    
    drop user vector cascade;
    
    create user vector identified by vector DEFAULT TABLESPACE tbs1 quota unlimited on tbs1;
    
    grant DB_DEVELOPER_ROLE to vector;
    
    create or replace directory VEC_DUMP as '/my_local_dir/';
    
    grant read, write on directory vec_dump to vector;
    ```

3.  Load your embedding model into the Oracle Database.

    Using the `DBMS_VECTOR` package, load your embedding model into the Oracle Database. You must specify the directory where you stored your model in ONNX format as well as describe what type of model it is and how you want to use it.

    For more information about downloading pretrained embedding models, converting them into ONNX format, and importing the ONNX file into Oracle Database, see [Import Pretrained Models in ONNX Format for Vector Generation Within the Database](GUID-D8140BF9-08E9-4B3F-9E28-E40A6FD181A4.md#).

    ```
    connect vector/<vector user password>@<pdb instance network name>
     
    exec dbms_vector.drop_onnx_model(model_name => 'doc_model', force => true);
     
    EXECUTE dbms_vector.load_onnx_model('VEC_DUMP', 'my_embedding_model.onnx', 'doc_model', JSON('{"function" : "embedding", "embeddingOutput" : "embedding" , "input": {"input": ["DATA"]}}'));
    
    ```

4.  Create a relational table to store books in the PDF format.

    You now create a table containing all the books you want to chunk and vectorize. You associate each new book with an ID and a pointer to your local directory where the books are stored.

    ```
    drop table documentation_tab purge;
    create table documentation_tab (id number, data blob);
    insert into documentation_tab values(1, to_blob(bfilename('VEC_DUMP', 'database-concepts23ai.pdf')));
    insert into documentation_tab values(2, to_blob(bfilename('VEC_DUMP', 'oracle-ai-vector-search-users-guide.pdf')));
    commit;
    select dbms_lob.getlength(data) from documentation_tab;
    ```

5.  Create a relational table to store unstructured data chunks and associated vector embeddings using `my_embedding_model.onnx`.

    You start by creating the table structure using the `VECTOR` data type. For more information about declaring a table's column as a `VECTOR` data type, see [Create Tables Using the VECTOR Data Type](GUID-E05AC257-CBD6-4B0C-A29F-0116EF02EA3A.md#).

    The `INSERT` statement reads each PDF file from `DOCUMENTATION_TAB`, transforms each PDF file into text, chunks each resulting text, then finally generates corresponding vector embeddings on each chunk that is created. All that is done in one single `INSERT` `SELECT` statement.

    Here you choose to use Vector Utility PL/SQL package `DBMS_VECTOR_CHAIN` to convert, chunk, and vectorize your unstructured data in one end-to-end pipeline. Vector Utility PL/SQL functions are intended to be a set of chainable stages \(using table functions\) through which you pass your input data to transform into a different representation. In this case, from PDF to text to chunks to vectors. For more information about using chainable utility functions in the `DBMS_VECTOR_CHAIN` package, see [About Chainable Utility Functions and Common Use Cases](GUID-63773EF5-071E-4C02-9EC9-D4E0BA2CE2A2.md#).

    ```
    drop table doc_chunks purge;
    create table doc_chunks (doc_id number, chunk_id number, chunk_data varchar2(4000), chunk_embedding vector);
     
    insert into doc_chunks
    select dt.id doc_id, et.embed_id chunk_id, et.embed_data chunk_data, to_vector(et.embed_vector) chunk_embedding
    from
        documentation_tab dt,
        dbms_vector_chain.utl_to_embeddings(
           dbms_vector_chain.utl_to_chunks(dbms_vector_chain.utl_to_text(dt.data), json('{"normalize":"all"}')),
           json('{"provider":"database", "model":"doc_model"}')) t,
        JSON_TABLE(t.column_value, '$[*]' COLUMNS (embed_id NUMBER PATH '$.embed_id', embed_data VARCHAR2(4000) PATH '$.embed_data', embed_vector CLOB PATH '$.embed_vector')) et;
     
    commit;
    ```

    **See Also:**

    -   [*Oracle Database JSON Developer’s Guide*](olink:ADJSN-GUID-0172660F-CE29-4765-BF2C-C405BDE8369A) for information about the `JSON_TABLE` function, which supports the `VECTOR` data type
6.  Generate a query vector for use in a similarity search.

    For a similarity search you will need query vectors. Here you enter your query text and generate an associated vector embedding.

    For example, you can use the following text: 'different methods of backup and recovery'. You use the `VECTOR_EMBEDDING` SQL function to generate the vector embeddings from the input text. The function takes an embedding model name and a text string to generate the corresponding vector. Note that you can generate vector embeddings outside of the database using your favorite tools. For more information about using the `VECTOR_EMBEDDING` SQL function, see [Use SQL Functions to Generate Embeddings](GUID-AEAA4AA9-2D47-4451-B6D5-E505C133D9CF.md#).

    In SQL\*Plus, use the following code:

    ```
    ACCEPT text_input CHAR PROMPT 'Enter text: '
    VARIABLE text_variable VARCHAR2(1000)
    VARIABLE query_vector VECTOR
    BEGIN
      :text_variable := '&text_input';
      SELECT vector_embedding(doc_model using :text_variable as data) into :query_vector;
    END;
    /
     
    PRINT query_vector
    ```

    In SQLCL, use the following code:

    ```
    DEFINE text_input = '&text'
     
    SELECT '&text_input';
     
    VARIABLE text_variable VARCHAR2(1000)
    VARIABLE query_vector CLOB
    BEGIN
      :text_variable := '&text_input';
      SELECT vector_embedding(doc_model using :text_variable as data) into :query_vector;
    END;
    /
     
    PRINT query_vector
    ```

7.  Run a similarity search to find, within your books, the first four most relevant chunks that talk about backup and recovery.

    Using the generated query vector, you search similar chunks in the `DOC_CHUNKS` table. For this, you use the `VECTOR_DISTANCE` SQL function and the `FETCH` SQL clause to retrieve the most similar chunks.

    For more information about the `VECTOR_DISTANCE` SQL function, see [Vector Distance Functions](GUID-8667D062-8084-4E55-8BD7-2C84E05F52FA.md#).

    For more information about exact similarity search, see [Perform Exact Similarity Search](GUID-CCCF06F5-AD46-466D-99B2-4609B84C2B69.md#).

    ```
    SELECT doc_id, chunk_id, chunk_data
    FROM doc_chunks
    ORDER BY vector_distance(chunk_embedding , :query_vector, COSINE)
    FETCH FIRST 4 ROWS ONLY;
    ```

    You can also add a `WHERE` clause to further filter your search, for instance if you only want to look at one particular book.

    ```
    SELECT doc_id, chunk_id, chunk_data
    FROM doc_chunks
    WHERE doc_id=1
    ORDER BY vector_distance(chunk_embedding , :query_vector, COSINE)
    FETCH FIRST 4 ROWS ONLY;
    ```

8.  Use the `EXPLAIN PLAN` command to determine how the optimizer resolves this query.

    ```
    EXPLAIN PLAN FOR
    SELECT doc_id, chunk_id, chunk_data
    FROM doc_chunks
    ORDER BY vector_distance(chunk_embedding , :query_vector, COSINE)
    FETCH FIRST 4 ROWS ONLY;
     
    select plan_table_output from table(dbms_xplan.display('plan_table',null,'all'));
    ```

    ```
    PLAN_TABLE_OUTPUT
    ------------------------------------------------------------------------------
    Plan hash value: 1651750914
    -------------------------------------------------------------------------------------------------
    | Id  | Operation               | Name          | Rows  | Bytes |TempSpc| Cost (%CPU)| Time     |
    -------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT        |               |     4 |   104 |       |   549   (3)| 00:00:01 |
    |*  1 |  COUNT STOPKEY          |               |       |       |       |            |          |
    |   2 |   VIEW                  |               |  5014 |   127K|       |   549   (3)| 00:00:01 |
    |*  3 |    SORT ORDER BY STOPKEY|               |  5014 |   156K|   232K|   549   (3)| 00:00:01 |
    |   4 |     TABLE ACCESS FULL   | DOC_CHUNKS    |  5014 |   156K|       |   480   (3)| 00:00:01 |
    -------------------------------------------------------------------------------------------------
    
    ```

9.  Run a multi-vector similarity search to find, within your books, the first four most relevant chunks in the first two most relevant books.

    Here you keep using the same query vector as previously used.

    For more information about performing multi-vector similarity search, see [Perform Multi-Vector Similarity Search](GUID-F5ACF1FB-87C8-4051-872D-63E408480EC8.md#).

    ```
    SELECT doc_id, chunk_id, chunk_data
    FROM doc_chunks
    ORDER BY vector_distance(chunk_embedding , :query_vector, COSINE)
    FETCH FIRST 2 PARTITIONS BY doc_id, 4 ROWS ONLY;
    ```

10. Create an In-Memory Neighbor Graph Vector Index on the vector embeddings that you created.

    When dealing with huge vector embedding spaces, you may want to create vector indexes to accelerate your similarity searches. Instead of scanning each and every vector embedding in your table, a vector index uses heuristics to reduce the search space to accelerate the similarity search. This is called approximate similarity search.

    For more information about creating vector indexes, see [Create Vector Indexes](GUID-8AF956F3-D951-4968-9B79-A6E180E87456.md#).

    **Note:** You must have explicit `SELECT` privilege to select from the `VECSYS.VECTOR$INDEX` table, which gives you detailed information about your vector indexes.

    ```
    create vector index docs_hnsw_idx on doc_chunks(chunk_embedding)
    organization inmemory neighbor graph
    distance COSINE
    with target accuracy 95;
     
    SELECT INDEX_NAME, INDEX_TYPE, INDEX_SUBTYPE
    FROM USER_INDEXES;
    INDEX_NAME     INDEX_TYPE  INDEX_SUBTYPE
    -------------- ----------- -----------------------------
    DOCS_HNSW_IDX  VECTOR      INMEMORY_NEIGHBOR_GRAPH_HNSW
    
    ...
     
    SELECT JSON_SERIALIZE(IDX_PARAMS returning varchar2 PRETTY)
    FROM VECSYS.VECTOR$INDEX where IDX_NAME = 'DOCS_HNSW_IDX';
    JSON_SERIALIZE(IDX_PARAMSRETURNINGVARCHAR2PRETTY)
    ________________________________________________________________
    {
      "type" : "HNSW",
      "num_neighbors" : 32,
      "efConstruction" : 300,
      "distance" : "COSINE",
      "accuracy" : 95,
      "vector_type" : "FLOAT32",
      "vector_dimension" : 384,
      "degree_of_parallelism" : 1,
      "pdb_id" : 3,
      "indexed_col" : "CHUNK_EMBEDDING"
    }
    ```

11. Determine the memory allocation in the vector memory area.

    To get an idea about the size of your In-Memory Neighbor Graph Vector Index in memory, you can use the `V$VECTOR_MEMORY_POOL` view. See [Size the Vector Pool](GUID-1815E227-56C9-4E62-977F-0FDA282C9D83.md#) for more information about sizing the vector pool to allow for vector index creation and maintenance.

    **Note:** You must have explicit `SELECT` privilege to select from the `V$VECTOR_MEMORY_POOL` view, which gives you detailed information about the vector pool.

    ```
    select CON_ID, POOL, ALLOC_BYTES/1024/1024 as ALLOC_BYTES_MB, 
    USED_BYTES/1024/1024 as USED_BYTES_MB
    from V$VECTOR_MEMORY_POOL order by 1,2;
    ```

12. Run an approximate similarity search to identify, within your books, the first four most relevant chunks.

    Using the previously generated query vector, you search chunks in the `DOC_CHUNKS` table that are similar to your query vector. For this, you use the `VECTOR_DISTANCE` function and the `FETCH APPROX` SQL clause to retrieve the most similar chunks using your vector index.

    For more information about approximate similarity search, see [Perform Approximate Similarity Search Using Vector Indexes](GUID-D8432ADA-38B0-4E5F-975F-E86977CA8488.md#).

    ```
    SELECT doc_id, chunk_id, chunk_data
    FROM doc_chunks
    ORDER BY vector_distance(chunk_embedding , :query_vector, COSINE)
    FETCH APPROX FIRST 4 ROWS ONLY WITH TARGET ACCURACY 80;
    ```

    You can also add a `WHERE` clause to further filter your search, for instance if you only want to look at one particular book.

    ```
    SELECT doc_id, chunk_id, chunk_data
    FROM doc_chunks
    WHERE doc_id=1
    ORDER BY vector_distance(chunk_embedding , :query_vector, COSINE)
    FETCH APPROX FIRST 4 ROWS ONLY WITH TARGET ACCURACY 80;
    ```

13. Use the `EXPLAIN PLAN` command to determine how the optimizer resolves this query.

    See [Optimizer Plans for Vector Indexes](GUID-80BBB84C-48F4-4C0B-9137-0B114D5515C9.md#) for more information about how the Oracle Database optimizer uses vector indexes to run your approximate similarity searches.

    ```
     
    EXPLAIN PLAN FOR
    SELECT doc_id, chunk_id, chunk_data
    FROM doc_chunks
    ORDER BY vector_distance(chunk_embedding , :query_vector, COSINE)
    FETCH APPROX FIRST 4 ROWS ONLY WITH TARGET ACCURACY 80;
     
    select plan_table_output from table(dbms_xplan.display('plan_table',null,'all'));
    ```

    ```
    PLAN_TABLE_OUTPUT
    --------------------------------------------------------------------------------
    Plan hash value: 2946813851
    --------------------------------------------------------------------------------------------------------
    | Id  | Operation                      | Name          | Rows  | Bytes |TempSpc| Cost (%CPU)| Time     |
    --------------------------------------------------------------------------------------------------------
    |   0 | SELECT STATEMENT               |               |     4 |   104 |       |   12083 (2)| 00:00:01 |
    |*  1 |  COUNT STOPKEY                 |               |       |       |       |            |          |
    |   2 |   VIEW                         |               |  5014 |   127K|       |   12083 (2)| 00:00:01 |
    |*  3 |    SORT ORDER BY STOPKEY       |               |  5014 |    19M|    39M|   12083 (2)| 00:00:01 |
    |   4 |     TABLE ACCESS BY INDEX ROWID| DOC_CHUNKS    |  5014 |    19M|       |    1    (0)| 00:00:01 |
    |   5 |      VECTOR INDEX HNSW SCAN    | DOCS_HNSW_IDX |  5014 |    19M|       |    1    (0)| 00:00:01 |
    --------------------------------------------------------------------------------------------------------
    
    ```

14. Determine your vector index performance for your approximate similarity searches.

    The index accuracy reporting feature allows you to determine the accuracy of your vector indexes. After a vector index is created, you may be interested to know how accurate your approximate vector searches are.

    The `DBMS_VECTOR.INDEX_ACCURACY_QUERY` PL/SQL procedure provides an accuracy report for a top-K index search for a specific query vector and a specific target accuracy. In this case you keep using the query vector generated previously. For more information about index accuracy reporting, see [Index Accuracy Report](GUID-A084929C-7A04-4764-9E5B-1204E0844CAF.md#).

    ```
    SET SERVEROUTPUT ON
    declare 
        report varchar2(128);
    begin 
        report := dbms_vector.index_accuracy_query(
            OWNER_NAME => 'VECTOR', 
            INDEX_NAME => 'DOCS_HNSW_IDX',
            qv => :query_vector,
            top_K => 10, 
            target_accuracy => 90 );
        dbms_output.put_line(report); 
    end; 
    /
    ```

    The report looks like the following: Accuracy achieved \(100%\) is 10% higher than the Target Accuracy requested \(90%\).


**Parent topic:**[Get Started](GUID-9FAD626E-42CA-4B9D-8CA5-618693D5ED5E.md)

