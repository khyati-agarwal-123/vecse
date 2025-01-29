## Convert Image to Embedding Using Public REST Providers {#GUID-1F06AAA4-7333-40F5-AF43-FE124A7378A4}

Perform an image-to-embedding transformation by making a REST call to the third-party service provider, Vertex AI. In this example, you can see how to vectorize both image and text inputs using a multimodal embedding model and then query a vector space containing vectors from both content types.

You can directly generate a vector embedding based on an image, which can be used for classifying or detecting images, comparing large datasets of images, or for performing a more effective similarity search on documents that include images. To get image embeddings, you can use any image embedding model or multimodal embedding model supported by Vertex AI. By analyzing an image, a model generates image embedding that encodes each visual element of the image (shape, color, pattern, texture, action, or object) as a vector representation.

Multimodal embedding is a technique that vectorizes data from different modalities such as text and images. This lets you use the same embedding model to generate embeddings for both types of content. By doing so, the resulting embeddings are compatible and situated in the same vector space, which allows for effective comparison between the two modalities (text and image) during similarity searches.

Here, you can use the `UTL_TO_EMBEDDING` function from either the `DBMS_VECTOR` or the `DBMS_VECTOR_CHAIN` package. 

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

To find a bird that is similar to a given image or text input, using similarity search:

  1. Connect as a local user and prepare your data dump directory.
    1. Log in to SQL*Plus as the `SYS` user, connecting as `SYSDBA`:
```
        conn sys/password as sysdba
```
```
        CREATE TABLESPACE tbs1
        DATAFILE 'tbs5.dbf' SIZE 20G AUTOEXTEND ON
        EXTENT MANAGEMENT LOCAL
        SEGMENT SPACE MANAGEMENT AUTO;
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
        

    2. Create a local user (`docuser`) and grant necessary privileges:
```
        drop user docuser cascade;
```
```
        create user docuser identified by docuser DEFAULT TABLESPACE tbs1 quota unlimited on tbs1;
```
```
        grant DB_DEVELOPER_ROLE to docuser;
```
        

    3. Create a local directory (`VEC_DUMP`) to store image files. Grant necessary privileges:
```
        create or replace directory VEC_DUMP as '/my_local_dir/';
```
```
        grant read, write on directory VEC_DUMP to docuser;
        
        commit;
```
        

    4. Connect as the local user (`docuser`):
```
        conn docuser/password
```
        

  2. Set the HTTP proxy server, if configured.
```
    EXEC UTL_HTTP.SET_PROXY(':');
```
    

  3. Grant connect privilege to `docuser` for allowing connection to the host, using the `DBMS_NETWORK_ACL_ADMIN` procedure.

This example uses `*` to allow any host. However, you can explicitly specify the host that you want to connect to. 
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
    

  4. Set up credentials for Vertex AI.

Vertex AI requires the following authentication parameter: 

`{ "access_token": "*``*" }`
```
    begin
    DBMS_VECTOR_CHAIN.DROP_CREDENTIAL(credential_name  => 'VERTEXAI_CRED');
    exception
    when others then null;
    end;
    /
```
    

Here, replace *``*  with your own value: 
```
    declare
    jo json_object_t;
    begin
    jo := json_object_t();
    jo.put('access_token', '');
    DBMS_VECTOR_CHAIN.CREATE_CREDENTIAL(
    credential_name   =>  'VERTEXAI_CRED',
    params            => json(jo.to_string));
    end;
    /
```
    

  5. Create a relational table (`docs`) and insert some images in it.

You will later compare your query image to this database of images. 

Here, you first create a `docs` table with `id` and `content` columns. Then, using the `to_blob` function, you convert the contents of your image files into `BLOB` for storing into `content` (blob column), reading the files from the `VEC_DUMP` directory. 
```
    drop table docs;
```
```
    create table docs(id number primary key, content blob);
    
    insert into docs(id, content)
    values(1, to_blob(bfilename('VEC_DUMP', 'cat.jpg')));
    insert into docs (id, content)
    values(2, to_blob(bfilename('VEC_DUMP', 'eagle.jpg')));
```
    

  6. Get image embedding for your query image (`parrots.jpg`):

First, upload your query image to the `VEC_DUMP` directory. 

Then, use `UTL_TO_EMBEDDING` to vectorize that image. Here, the input is specified as `parrots.jpg` (with a pointer to the `VEC_DUMP` directory), the modality is specified as `image`, and the provider-specific embedding parameters for Vertex AI are passed as JSON. 

> **note:** For a list of all supported REST endpoints, see [Supported Third-Party Provider Operations and Endpoints](supported-third-party-provider-operations-and-endpoints.md#GUID-BE3EE403-CD10-4708-A15F-EFB1FA69DF09). 
```
    -- declare embedding parameters
    
    var params clob;
    
    begin
    :params := '
    {
    "provider": "vertexai",
    "credential_name": "VERTEXAI_CRED",
    "url": "https://LOCATION-aiplatform.googleapis.com/v1/projects/PROJECT/locations/LOCATION/publishers/google/models/",
    "model": "multimodalembedding:predict"
    }';
    end;
    /
    
    -- get image embedding: PL/SQL example
    
    declare
    v vector;
    output clob;
    begin
    v := dbms_vector_chain.utl_to_embedding(
    to_blob(bfilename('VEC_DUMP', 'parrots.jpg')), 'image', json(:params));
    output := vector_serialize(v);
    dbms_output.put_line('vector data=' || dbms_lob.substr(output, 100) || '...');
    end;
    /
    
    -- get image embedding: select example
    
    select dbms_vector_chain.utl_to_embedding(
    to_blob(bfilename('VEC_DUMP', 'parrots.jpg')), 'image', json(:params));
```
    

  7. Search using an image input. Here, input is the generated embedding for your query image (`parrots.jpg`):
```
    select docs.id, vector_distance(
    dbms_vector_chain.utl_to_embedding(to_blob(bfilename('VEC_DUMP', 'parrots.jpg')), 'image', json(:params)),
    dbms_vector_chain.utl_to_embedding(docs.content, 'image', json(:params)),
    cosine) dist
    from docs
    order by dist asc;
```
    

An output appears as:
```
    ID       DIST
    -------  -----------
    2        5.847E-001
    1        6.295E-001
```
    

This query shows that `ID 2` (`eagle.jpg`) is the closest match to the given image input, while `ID 1` (`cat.jpg`) is less similar. 

  8. Search using a text input. Here, input is the image's description as "`bald eagle`":
```
    select docs.id, vector_distance(
    dbms_vector_chain.utl_to_embedding('bald eagle', json(:params)),
    dbms_vector_chain.utl_to_embedding(docs.content, 'image', json(:params)),
    cosine) dist
    from docs
    order by dist asc;
```
    

An output appears as:
```
    ID      DIST
    ------- -----------
    2       8.449E-001
    1       9.87E-001
```
    

This query also implies that `ID 2` is the closest match to your text input, while `ID 1` is less similar. However, both values are lower than those in the previous query where you specified the image as input. 




**Related Topics**

  * [UTL_TO_EMBEDDING and UTL_TO_EMBEDDINGS](utl_to_embedding-and-utl_to_embeddings-dbms_vector_chain.md#GUID-C6439E94-4E86-4ECD-954E-4B73D53579DE)
  * [DBMS_VECTOR](dbms_vector-vecse.md#GUID-829230F9-BD1E-41F9-BAAB-5D3C3E52FC12)
  * [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md#GUID-A09FF69E-FCCB-4EDA-B7E4-B02A11359504)



**Parent topic:** [Generate Embeddings](generate-embeddings.md)
