---
author: [Yuan Zhou, Weiwei Gong, Vinita Subramanian, Teck Hua Lee, Tirthankar Lahiri, Shasank Chavan, Sebastian DeLaHoz, Roger Ford, Rohan Aggarwal, Mark Hornick, Malavika S P, Harichandan Roy, George Krupka, Doug Hood, Dinesh Das, David Jiang, Boriana Milenova, Bonnie Xia, Aurosish Mishra, Angela Amor, Agnivo Saha, Aleksandra Czarlinska, Ramya P, Usha Krishnamurthy, Tulika Das, Suresh Rajan, Sarika Surampudi, Sarah Hirschfeld, Prakash Jashnani, Jody Glover, Jessica True, Mamata Basapur, Maitreyee Chaliha, Gunjan Jain, Frederick Kush, Douglas Williams, Binika Kumar, Jean-Francois Verrier]
publisherinformation: May2024
---

# Convert Text String to Embedding

You can vectorize text strings like this for chatbots or information-retrieval applications, where you want to directly convert a user's input text to a query vector and then run it against vector index for a fast similarity search.

You can perform a text-to-embedding transformation using the `UTL_TO_EMBEDDING` PL/SQL API \(note the singular "embedding"\) or the `VECTOR_EMBEDDING` SQL function. Both `UTL_TO_EMBEDDING` and `VECTOR_EMBEDDING` directly return a `VECTOR` type \(not an array\).

Determine which API to use:

-   If you want to access a third-party embedding model, then you can use `UTL_TO_EMBEDDING` from either the `DBMS_VECTOR` or `DBMS_VECTOR_CHAIN` package.

    This scenario uses the `DBMS_VECTOR.UTL_TO_EMBEDDING` API.

-   If you are using an ONNX format embedding model, then you can use both `VECTOR_EMBEDDING` and `UTL_TO_EMBEDDING`.


To generate a vector embedding with "`hello`" as the input:

1.  Start SQL\*Plus and connect to Oracle Database as a local test user.

    1.  Log in to SQL\*Plus as the `sys` user, connecting as `sysdba`, to a pluggable database \(PDB\) within your multitenant container database \(CDB\):

        ```
        conn sys/password@CDB_PDB as sysdba
        ```

        ```
        CREATE TABLESPACE tbs1
        DATAFILE 'tbs5.dbf' SIZE 20G AUTOEXTEND ON
        EXTENT MANAGEMENT LOCAL
        SEGMENT SPACE MANAGEMENT AUTO;
        ```

    2.  Create a local test user \(`docuser`\) and grant necessary privileges:

        ```
        DROP USER docuser cascade;
        ```

        ```
        CREATE USER docuser identified by docuser DEFAULT TABLESPACE tbs1 quota unlimited on tbs1;
        ```

        ```
        GRANT DB_DEVELOPER_ROLE, create credential to docuser;
        ```

    3.  Connect to Oracle Database as the test user and alter the environment settings for your session:

        ```
        CONN docuser/password@CDB_PDB
        ```

        ```
        SET ECHO ON
        SET FEEDBACK 1
        SET NUMWIDTH 10
        SET LINESIZE 80
        SET TRIMSPOOL ON
        SET TAB OFF
        SET PAGESIZE 10000
        SET LONG 10000
        ```

    4.  Set the HTTP proxy server, if configured:

        ```
        EXEC UTL_HTTP.SET_PROXY('<proxy-hostname\>:<proxy-port\>');
        ```

    5.  Grant connect privilege for a host using the `DBMS_NETWORK_ACL_ADMIN` procedure. This example uses `*` to allow any host. However, you can explicitly specify each host that you want to connect to.

        ```
        BEGIN
          DBMS_NETWORK_ACL_ADMIN.APPEND_HOST_ACE(
            host => '*',
            ace => xs$ace_type(privilege_list => xs$name_list('connect'),
                               principal_name => 'docuser',
                               principal_type => xs_acl.ptype_db));
        END;
        /
        ```

2.  If you are using a third-party embedding model and need to make a REST call, set up your credentials for the REST provider and then call `UTL_TO_EMBEDDING`.

    -   **Using Cohere, Google AI, Hugging Face, OpenAI, and Vertex AI**:

        1.  Run `DBMS_VECTOR.CREATE_CREDENTIAL` to create and store a credential.

            Cohere, Google AI, Hugging Face, OpenAI, and Vertex AI require the following authentication parameter:

            `{ "access_token": "<access token\>" }`

            You will later refer to this credential name when declaring JSON parameters for the `UTL_to_EMBEDDING` call.

            ```
            exec dbms_vector.drop_credential('<credential name\>');
            ```

            ```
            declare
              jo json_object_t;
            begin
              jo := json_object_t();
              jo.put('access_token', '<access token\>');
              dbms_vector.create_credential(
                credential_name   => '<credential name\>',
                params            => json(jo.to_string));
            end;
            /
            ```

            Replace the `access_token` and `credential_name` values. For example:

            ```
            declare
              jo json_object_t;
            begin
              jo := json_object_t();
              jo.put('access_token', 'AbabA1B123aBc123AbabAb123a1a2ab');
              dbms_vector.create_credential(
                credential_name   => 'HF_CRED',
                params            => json(jo.to_string));
            end;
            /
            ```

        2.  Call `DBMS_VECTOR.UTL_TO_EMBEDDING`:

            ```
            *-- select example*
            
            var params clob;
            exec :params := '
            {
              "provider": "<REST provider\>",
              "credential_name": "<credential name\>",
              "url": "<REST endpoint URL for embedding service\>",
              "model": "<embedding model name\>"
            }';
            
            select dbms_vector.utl_to_embedding('hello', json(:params)) from dual;
            
            *-- PL/SQL example*
            
            declare
              input clob;
              params clob;
              v vector;
            begin
              input := 'hello';
            
              params := '
            {
              "provider": "<REST provider\>",
              "credential_name": "<credential name\>",
              "url": "<REST endpoint URL for embedding service\>",
              "model": "<embedding model name\>"
            }';
            
              v := dbms_vector.utl_to_embedding(input, json(params));
              dbms_output.put_line(vector_serialize(v));
            exception
              when OTHERS THEN
                DBMS_OUTPUT.PUT_LINE (SQLERRM);
                DBMS_OUTPUT.PUT_LINE (SQLCODE);
            end;
            /
            ```

            Replace the `provider`, `credential_name`, `url`, and `model` values. Optionally, you can specify additional REST provider parameters.

            Cohere example:

            ```
            {
              "provider": "cohere",
              "credential_name": "COHERE_CRED",
              "url": "https://api.cohere.example.com/embed",
              "model": "embed-model",
              "input_type": "search_query"
            }
            ```

            Google AI example:

            ```
            {
              "provider": "googleai",
              "credential_name": "GOOGLEAI_CRED",
              "url": "https://googleapis.example.com/models/",
              "model": "embed-model"
            }
            ```

            Hugging Face example:

            ```
            {
              "provider": "huggingface",
              "credential_name": "HF_CRED",
              "url": "https://api.huggingface.example.com/",
              "model": "embed-model",
              "wait_for_model": "true"
            }
            ```

            OpenAI example:

            ```
            {
              "provider": "openai",
              "credential_name": "OPENAI_CRED",
              "url": "https://api.openai.example.com/embeddings",
              "model": "embed-model"
            }
            ```

            Vertex AI example:

            ```
            {
              "provider": "vertexai",
              "credential_name": "VERTEXAI_CRED",
              "url": "https://googleapis.example.com/models/",
              "model": "embed-model"
            }
            ```

    -   **Using Generative AI**:

        1.  Run `DBMS_VECTOR.CREATE_CREDENTIAL` to create and store an OCI credential \(`OCI_CRED`\).

            Generative AI requires the following authentication parameters:

            ```
            { 
            "user_ocid": "<user ocid\>",
            "tenancy_ocid": "<tenancy ocid\>",
            "compartment_ocid": "<compartment ocid\>",
            "private_key": "<private key\>",
            "fingerprint": "<fingerprint\>" 
            }
            ```

            You will later refer to this credential name when declaring JSON parameters for the `UTL_to_EMBEDDING` call.

            **Note:**

            The generated private key may appear as:

            ```
            -----BEGIN RSA PRIVATE KEY-----
            <private key string\>
            -----END RSA PRIVATE KEY-----
            ```

            You pass the `<private key string\>` value \(excluding the `BEGIN` and `END` lines\), either as a single line or as multiple lines.

            ```
            exec dbms_vector.drop_credential('OCI_CRED');
            ```

            ```
            declare
              jo json_object_t;
            begin
              jo := json_object_t();
              jo.put('user_ocid','<user ocid\>');
              jo.put('tenancy_ocid','<tenancy ocid\>');
              jo.put('compartment_ocid','<compartment ocid\>');
              jo.put('private_key','<private key\>');
              jo.put('fingerprint','<fingerprint\>');
              dbms_output.put_line(jo.to_string);
              dbms_vector.create_credential(
                credential_name   => 'OCI_CRED',
                params            => json(jo.to_string));
            end;
            /
            ```

            Replace all the authentication parameter values. For example:

            ```
            declare
              jo json_object_t;
            begin
              jo := json_object_t();
              jo.put('user_ocid','ocid1.user.oc1..aabbalbbaa1112233aabbaabb1111222aa1111bb');
              jo.put('tenancy_ocid','ocid1.tenancy.oc1..aaaaalbbbb1112233aaaabbaa1111222aaa111a');
              jo.put('compartment_ocid','ocid1.compartment.oc1..ababalabab1112233abababab1111222aba11ab');
              jo.put('private_key','AAAaaaBBB11112222333...AAA111AAABBB222aaa1a/+');
              jo.put('fingerprint','01:1a:a1:aa:12:a1:12:1a:ab:12:01:ab:a1:12:ab:1a');
              dbms_output.put_line(jo.to_string);
              dbms_vector.create_credential(
                credential_name   => 'OCI_CRED',
                parameters        => json(jo.to_string));
            end;
            /
            ```

        2.  Call `DBMS_VECTOR.UTL_TO_EMBEDDING`:

            ```
            *-- select example*
            
            var params clob;
            exec :params := '
            {
              "provider": "ocigenai",
              "credential_name": "OCI_CRED",
              "url": "<REST endpoint URL for embedding service\>",
              "model": "<REST provider embedding model name\>"
            }';
            
            select dbms_vector.utl_to_embedding('hello', json(:params)) from dual;
            
            *-- PL/SQL example*
            
            declare
              input clob;
              params clob;
              v vector;
            begin
              input := 'hello;
            
              params := '
            {
              "provider": "ocigenai",
              "credential_name": "OCI_CRED",
              "url": "<REST endpoint URL for embedding service\>",
              "model": "<REST provider embedding model name\>"
            }';
            
              v := dbms_vector.utl_to_embedding(input, json(params));
              dbms_output.put_line(vector_serialize(v));
            exception
              when OTHERS THEN
                DBMS_OUTPUT.PUT_LINE (SQLERRM);
                DBMS_OUTPUT.PUT_LINE (SQLCODE);
            end;
            /
            ```

            Replace the `url` and `model` values. Optionally, you can specify additional REST provider-specific parameters.

            For example:

            ```
            {
              "provider": "ocigenai",
              "credential_name": "OCI_CRED",
              "url": "https://generativeai.oci.example.com/embedText",
              "model": "embed-modelname",
              "batch_size": 10
            }
            ```

3.  If you are using a declared embedding model, then call either `VECTOR_EMBEDDING` or `UTL_TO_EMBEDDING`.

    1.  Load your ONNX model into Oracle Database.

        For detailed information on how to perform this step, see [Import ONNX Models and Generate Embeddings](GUID-6AEA7A0E-78E0-4083-A126-4516EB98175A.md#).

        Here, `doc_model` specifies the name under which the imported model is stored in Oracle Database.

    2.  Call `VECTOR_EMBEDDING` or `UTL_TO_EMBEDDING`:

        -   `VECTOR_EMBEDDING`:

            ```
            SELECT TO_VECTOR(VECTOR_EMBEDDING(doc_model USING 'hello' as data)) AS embedding;
            ```

        -   `DBMS_VECTOR.UTL_TO_EMBEDDING`:

            ```
            var params clob; exec :params := '{"provider":"database", "model":"doc_model"}';
            
            select dbms_vector.utl_to_embedding('hello', json(:params)) from dual;
            ```

    The generated embedding appears as follows:

    ```
    EMBEDDING
    -------------------------------------------------------------------------------------------------------------------------------------
    [8.78423732E-003,-4.29633334E-002,-5.93001908E-003,-4.65480909E-002,2.14333013E-002,6.53376281E-002,-5.93746938E-002,2.10403297E-002,
    4.38376889E-002,5.22960871E-002,1.25104953E-002,6.49512559E-002,-9.26998071E-003,-6.97442219E-002,-3.02916039E-002,-4.74979728E-003,
    -1.08755399E-002,-4.63751052E-003,3.62781435E-002,-9.35919806E-002,-1.13934642E-002,-5.74270077E-002,-1.36667723E-002,2.42995787E-002,
    -6.96804151E-002,4.93822657E-002,1.01460628E-002,-1.56464987E-002,-2.39410568E-002,-4.27529104E-002,-5.65665103E-002,-1.74160264E-002,
    5.05326502E-002,4.31500375E-002,-2.6994409E-002,-1.72731467E-002,9.30535868E-002,6.85951149E-004,5.61876409E-003,-9.0233935E-003,
    -2.55788807E-002,-2.04174276E-002,3.74175981E-002,-1.67872179E-002,1.07479304E-001,-6.64602639E-003,-7.65537247E-002,-9.71965566E-002,
    -3.99636962E-002,-2.57076006E-002,-5.62455431E-002,-1.3583754E-001,3.45946029E-002,1.85191762E-002,3.01524661E-002,-2.62163244E-002,
    -4.05582506E-003,1.72979087E-002,-3.66434865E-002,-1.72491539E-002,3.95228416E-002,-1.05518714E-001,-1.27463877E-001,1.42578809E-002
    ```


This example uses the default settings for each provider. For detailed information on additional parameters, refer to your third-party provider's documentation.

**Parent topic:**[Generate Embeddings: SQL and PL/SQL Examples](GUID-813E0E54-9EEF-43FA-A506-1F276D47E7A6.md)

**Related information**  


[UTL\_TO\_EMBEDDING](olink:ARPLS-GUID-8E615832-F6C0-4435-8F43-3FAF80692D5B)

[VECTOR\_EMBEDDING](olink:SQLRF-GUID-5ED78260-6D21-4B6B-86E0-A1E70EFA11CA)

