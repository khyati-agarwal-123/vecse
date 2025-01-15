##  Import Pretrained Models in ONNX Format for Vector Generation Within the Database {#GUID-D8140BF9-08E9-4B3F-9E28-E40A6FD181A4} 

You can download pretrained embedding machine learning models, convert them into ONNX format if they are not already in ONNX format, import the ONNX format models into Oracle Database, and generate vector embeddings from your data within the database. 

  * [ Convert Pretrained Models to ONNX Format ](convert-trained-models-onnx-format.md)   
OML4Py enables the use of text transformers from Hugging Face by converting them into ONNX format models. OML4Py also adds the necessary tokenization and post-processing. The resulting ONNX pipeline is then imported into the database and can be used to generate embeddings for AI Vector Search. 
  * [ Import ONNX Models into Oracle Database End-to-End Example ](import-onnx-models-oracle-database-end-end-example.md)   
Learn to import a pretrained embedding model that is in ONNX format and generate vector embeddings. 



**Parent topic:** [ Generate Vector Embeddings ](generate-vector-embeddings-node.md)
