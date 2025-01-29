## CREATE_CREDENTIAL {#GUID-A6E28402-DC43-44C6-A1B2-75C3F270DD76}

Use the `DBMS_VECTOR_CHAIN.CREATE_CREDENTIAL` credential helper procedure to create a credential name for storing user authentication details in Oracle Database. 

Purpose

To securely manage authentication credentials in the database. You require these credentials to enable access during REST API calls to your chosen third-party service provider, such as Cohere, Google AI, Hugging Face, Oracle Cloud Infrastructure (OCI) Generative AI, OpenAI, or Vertex AI.

A credential name holds authentication parameters, such as user name, password, access token, private key, or fingerprint.

Note that if you are using Oracle Database as the service provider, then you do not need to create a credential.

> **note:** WARNING: 

Certain features of the database may allow you to access services offered separately by third-parties, for example, through the use of JSON specifications that facilitate your access to REST APIs. 

Your use of these features is solely at your own risk, and you are solely responsible for complying with any terms and conditions related to use of any such third-party services. Notwithstanding any other terms and conditions related to the third-party services, your use of such database features constitutes your acceptance of that risk and express exclusion of Oracle's responsibility or liability for any damages resulting from such access.

Syntax
```
    DBMS_VECTOR_CHAIN.CREATE_CREDENTIAL (
    CREDENTIAL_NAME     IN VARCHAR2,
    PARAMS              IN JSON DEFAULT NULL
    );
```
    

CREDENTIAL_NAME

Specify a name of the credential that you want to create for holding authentication parameters.

PARAMS

Specify authentication parameters in JSON format, based on your chosen service provider.

Generative AI requires the following authentication parameters: 
```
    {
    "user_ocid"       : "",
    "tenancy_ocid"    : "",
    "compartment_ocid": "",
    "private_key"     : "",
    "fingerprint"     : ""
    }
```
    

Cohere, Google AI, Hugging Face, OpenAI, and Vertex AI require the following authentication parameter: 
```
    { "access_token": "" }
```
    

**Table: Parameter Details**

Parameter | Description  
---|---  
`user_ocid` |  Oracle Cloud Identifier (OCID) of the user, as listed on the User Details page in the OCI console.  
`tenancy_ocid` |  OCID of your tenancy, as listed on the Tenancy Details page in the OCI console.  
`compartment_ocid` |  OCID of your compartment, as listed on the Compartments information page in the OCI console.  
`private_key` |  OCI private key. **Note**: <br>The generated private key may appear as: <br>```-----BEGIN RSA PRIVATE KEY----- -----END RSA PRIVATE KEY-----```<br>You pass the *``*  value (excluding the `BEGIN` and `END` lines), either as a single line or as multiple lines.   
`fingerprint` |  Fingerprint of the OCI profile key, as listed on the User Details page under API Keys in the OCI console.  
`access_token` |  Access token obtained from your third-party service provider.  
  
Required Privilege

You need the `CREATE CREDENTIAL` privilege to call this API. 

Examples

  * For Generative AI:
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
    dbms_vector_chain.create_credential(
    credential_name   => 'OCI_CRED',
    params            => json(jo.to_string));
    end;
    /
```
    

  * For Cohere:
```
    declare
    jo json_object_t;
    begin
    jo := json_object_t();
    jo.put('access_token', 'A1Aa0abA1AB1a1Abc123ab1A123ab123AbcA12a');
    dbms_vector_chain.create_credential(
    credential_name   => 'COHERE_CRED',
    params            => json(jo.to_string));
    end;
    /
```
    




**End-to-end examples**: 

To run end-to-end example scenarios using this procedure, see [Use LLM-Powered APIs to Generate Summary and Text](https://docs.oracle.com/pls/topic/lookup?ctx=en/database/oracle/oracle-database/23/vecse&id=VECSE-GUID-D6F4B046-2989-4E0E-9245-C56F39866DA1). 

**Parent topic:** [DBMS_VECTOR_CHAIN](dbms_vector_chain-vecse.md)
