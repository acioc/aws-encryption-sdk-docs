# Using Keyrings<a name="choose-keyring"></a>

The AWS Encryption SDK for C and AWS Encryption SDK for JavaScript use *keyrings* to perform [envelope encryption](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-envelope-encryption)\. Keyrings generate, encrypt, and decrypt data keys\. The keyrings that you use determines the source of the unique data keys that protect each message, and the wrapping keys that encrypt that data key\. You specify a keyring when encrypting and the same or a different keyring when decrypting\. You can use the keyrings that the SDK provides or write your own compatible custom keyrings\. 

You can use each keyring individually or combine keyrings into a [multi\-keyring](#use-multi-keyring)\. Although most keyrings can generate, encrypt, and decrypt data keys, you might create a keyring that performs only one particular operation, such as a keyring that only generates data keys, and use that keyring in combination with others\.

We recommend that you use a keyring that protects your master keys and performs cryptographic operations within a secure boundary, such as the KMS keyring, which uses [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) [customer master keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMKs\) that never leave AWS KMS unencrypted\. You can also write a keyring that uses master keys that are stored in your hardware security modules \(HSMs\) or protected by other master key services\. 

This topic explains how to use the keyring feature of the AWS Encryption SDK and how to choose a keyring\. For examples of creating and using keyrings, see the [C](c-language.md) and [JavaScript](javascript.md) topics\.

**Topics**
+ [How Keyrings Work](#using-keyrings)
+ [Keyring Compatibility](#keyring-compatibility)
+ [KMS Keyrings](#use-kms-keyring)
+ [Raw AES Keyrings](#use-raw-aes-keyring)
+ [Raw RSA Keyrings](#use-raw-rsa-keyring)
+ [Multi\-Keyrings](#use-multi-keyring)

## How Keyrings Work<a name="using-keyrings"></a>

One of the most important tasks you perform when using the AWS Encryption SDK for C or JavaScript is selecting and configuring a keyring\. A [keyring](concepts.md#keyring) generates, encrypts, and decrypts data keys\. Each keyring is typically associated with a wrapping key or a service that provides and protects wrapping keys\. You can use the keyrings that the AWS Encryption SDK provides or write your own compatible custom keyrings\. For help in choosing a keyring, see the following sections that describe each keyring\.

Keyrings make it easier for you to determine which wrapping keys are used to encrypt and decrypt your data\. Keyrings take the place of master key providers in the Java and Python implementations of the AWS Encryption SDK\. Despite this architectural difference, all of the language implementations are fully interoperable, subject to language constraints\. However, you must configure the keyring and master key provider with the same or corresponding wrapping keys\. For details, see [Keyring Compatibility](#keyring-compatibility)\.

You instantiate and configure your keyring, but you don't interact with it directly\. The [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\) interacts with the keyring for you\. 

When you encrypt data, the CMM asks the keyring for encryption materials\. The keyring returns a plaintext key and a copy of the key that's encrypted by each of the wrapping keys in the keyring\. The AWS Encryption SDK uses the plaintext key to encrypt the data, and stores the encrypted data keys with the data in the [encrypted message](concepts.md#message) that it returns\.

![\[Encrypting with a CMM and keyring. The image shows a keyring that contains three master keys. When the CMM asks the keyring for encryption materials, the keyring returns a unique plaintext data key and three encrypted copies of that data key, one encrypted by each of the three master keys.\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/keyring-encrypt.png)

When you decrypt data, the CMM passes in the encryption keys from the encrypted message and asks the keyring to decrypt any one of them\. The keyring uses its wrapping keys to decrypt one of the encrypted data keys and returns a plaintext data key\. The AWS Encryption SDK uses the plaintext data key to decrypt the data\. If none of the wrapping keys in the keyring can decrypt any of the encrypted data keys, the decrypt operation fails\.

![\[Decrypting with a CMM and keyring. The image shows a keyring that contains three master keys. When the CMM asks the keyring for decryption materials and passes in three encrypted data keys, the keyring uses its master keys to decrypt one of the data keys. Then it returns the plaintext data key to the CMM.\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/keyring-decrypt.png)

You can use a single keyring or also combine keyrings of the same type or a different type into a [multi\-keyring](#use-multi-keyring)\. When you encrypt data, the multi\-keyring returns a copy of the data key encrypted by all of the wrapping keys in all of the keyrings that comprise the multi\-keyring\. You can decrypt the data using a keyring configured with any one of the wrapping keys in the multi\-keyring\.

## Keyring Compatibility<a name="keyring-compatibility"></a>

Although the Java, Python, C, and JavaScript implementations of the AWS Encryption SDK have some architectural differences, they are designed to be fully compatible, subject to language constraints\. You can encrypt your data using one programming language implementation and decrypt it with any other language implementation\. However, you must use the same or corresponding master keys to encrypt and decrypt your data keys\. For information about language constraints, see the topic about each language implementation, such as [Compatibility of the AWS Encryption SDK for JavaScript](javascript.md#javascript-compatibility) in the AWS Encryption SDK for JavaScript topic\.

The following table shows which master keys and master key providers are compatible with the keyrings provided in the AWS Encryption SDK for C and the AWS Encryption SDK for JavaScript\. Any minor incompatibility due to language constraints is explained in the topic about the language implementation\.


**Compatible Keyrings and Master Key Providers**  

| Keyring: C and JavaScript | Master Key Provider: Java and Python | 
| --- | --- | 
| [KMS keyring](#use-kms-keyring) |  [KMS master key \(Java\)](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/kms/KmsMasterKey.html) [KMS master key provider \(Java\)](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/kms/KmsMasterKeyProvider.html) [KMS master key \(Python\)](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.kms.html) [KMS master key provider \(Python\)](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.kms.html#aws_encryption_sdk.key_providers.kms.KMSMasterKeyProvider)  | 
| [Raw AES keyring](#use-raw-aes-keyring) | When they are used with symmetric encryption keys:[JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) \(Java\)[RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) \(Python\) | 
| [Raw RSA keyring](#use-raw-rsa-keyring) | When they are used with asymmetric encryption keys:[JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) \(Java\)[RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) \(Python\) | 

When decrypting, the Java and Python implementations behave like the [KMS Discovery keyring](#kms-keyring-discovery), that is, they don't limit the CMKs that can be used to decrypt the encrypted data keys\. Also, the Java and Python SDKs don't supply an equivalent for the [KMS Regional Discovery keyring](#kms-keyring-regional), but you can create your own custom keyrings\. 

## KMS Keyrings<a name="use-kms-keyring"></a>

A KMS keyring uses AWS Key Management Service \(AWS KMS\) [customer master keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMKs\) to generate, encrypt, and decrypt data keys\. AWS KMS protects your master keys and performs cryptographic operations within the FIPS boundary\. We recommend that you use a KMS keyring, or a keyring with similar security properties, whenever possible\.

A KMS keyring can have a *generator key*, which is the CMK that generates the plaintext data key that protects your data and encrypts it\. It can also have additional CMKs that encrypt the same plaintext data key\. When encrypting, the KMS keyring that you use must have a generator key\. Generator keys are not required for decrypting\. When decrypting, any key in the KMS keyring can be used to decrypt an encrypted data key\.

Like all keyrings, KMS keyrings can be used independently or in a [multi\-keyring](#use-multi-keyring) with other keyrings of the same or a different type\.

**Topics**
+ [Required Permissions for KMS Keyrings](#kms-keyring-permissions)
+ [Encrypting with a KMS Keyring](#kms-keyring-encrypt)
+ [Decrypting with a KMS Keyring](#kms-keyring-decrypt)
+ [Using a KMS Discovery Keyring](#kms-keyring-discovery)
+ [Using a KMS Regional Discovery Keyring](#kms-keyring-regional)

### Required Permissions for KMS Keyrings<a name="kms-keyring-permissions"></a>

The AWS Encryption SDK doesn't require an AWS account and it doesn't depend on any AWS service\. However, to use an AWS Key Management Service \(AWS KMS\) [customer master key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) in a KMS keyring, you need an AWS account and the following minimum permissions on the AWS KMS customer master keys in your keyring\. 
+ To encrypt with a KMS keyring, you need [kms:GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) permission on the generator key\. You need [kms:Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) permission on all additional keys in the KMS keyring\.
+ To decrypt with a KMS keyring, you need [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) permission on at least one key in the KMS keyring\.
+ To encrypt with a multi\-keyring comprised of KMS keyrings, you need [kms:GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) permission on the generator key in the generator keyring\. You need [kms:Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) permission on all other keys in all other KMS keyrings\. 

For detailed information about permissions for AWS KMS customer master keys, see [Authentication and Access Control for AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/control-access.html) in the *AWS Key Management Service Developer Guide*\.

### Encrypting with a KMS Keyring<a name="kms-keyring-encrypt"></a>

You can configure each KMS keyring with a single CMK or multiple CMKs in the same or different AWS accounts and AWS Regions\. You can also configure a [KMS discovery keyring](#kms-keyring-discovery) with no CMKs\. As with all keyrings, you can use one or more KMS keyrings in a [multi\-keyring](#use-multi-keyring)\. 

When you create a KMS keyring to encrypt data, you must specify a *generator key*, which is a CMK that generates a plaintext data key and encrypts it\. Then, if you choose, you can specify additional CMKs that encrypt the same plaintext data key\. The [encrypted message](concepts.md#message) that the AWS Encryption SDK returns includes the ciphertext and all of the encrypted data keys\. The caller must have `kms:GenerateDataKey` permission for the generator CMK, and `kms:Encrypt` permission for all additional CMKs\.

For example, the following KMS keyring specifies a generator key and one additional key\. When you use this keyring to encrypt data, it returns one plaintext data key generated by the generator key and two encrypted data keys, one encrypted under the generator key and one encrypted under the additional key\. To decrypt the data protected by this keyring, the keyring that you use must include at least one of the CMKs that encrypted the data key, or no CMKs\. \(A KMS keyring with no CMKs is known as a [discovery keyring](#kms-keyring-discovery)\.\) 

**Note**  
If you plan to use the same keyring for encrypting and decrypting data, use a key ARN to identify each CMK\. Key ARNs are required when decrypting\.

------
#### [ C ]

When you specify an AWS KMS CMK for a keyring in the AWS Encryption SDK for C, you must use the Amazon Resource Name \(ARN\) of the CMK\. In an encryption keyring, you can specify the key ARN or alias ARN\. In a decryption keyring, you must use the key ARN\. For help finding the key ARN, see [Finding the Key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\. To find the alias ARN, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) API\.

For a complete example, see [string\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/string.cpp)\.

```
const char * generator_key = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
const char * additional_key = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"    

struct aws_cryptosdk_keyring *kms_encrypt_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(generator_key,{additional_key});
```

------
#### [ JavaScript Browser ]

When you specify an AWS KMS CMK for an encryption keyring in the AWS Encryption SDK for JavaScript, you can use any valid CMK identifier: a key ID, alias name, key ARN, or alias ARN\. For help finding the key ARN, see [Finding the Key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\. To find the alias ARN, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) API\.

For a complete example, see [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/kms_simple.ts)\.

```
const clientProvider = getClient(KMS, { credentials })
const generatorKeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
const additionalKey = 'alias/exampleAlias'

const keyring = new KmsKeyringBrowser({
  clientProvider, 
  generatorKeyId, 
  keyIds: [additionalKey] 
})
```

------
#### [ JavaScript Node\.js ]

When you specify an AWS KMS CMK for an encryption keyring in the AWS Encryption SDK for JavaScript, you can use any valid CMK identifier: a key ID, alias name, key ARN, or alias ARN\. For help finding the key ARN, see [Finding the Key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\. To find the alias ARN, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) API\.

For a complete example, see [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_simple.ts)\.

```
const generatorKeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
const additionalKey = 'alias/exampleAlias'

const keyring = new KmsKeyringNode({
  generatorKeyId,
  keyIds: [additionalKey]
})
```

------

### Decrypting with a KMS Keyring<a name="kms-keyring-decrypt"></a>

You also specify a KMS keyring when decrypting the [encrypted message](concepts.md#message) that the AWS Encryption SDK returns\. You can use a decryption keyring to determine which encrypted data keys can be decrypted\. If the decryption keyring includes CMKs, the AWS Encryption SDK will only decrypt encrypted data keys that were encrypted by the CMKs in the keyring\. \(You can also use a [discovery keyring](#kms-keyring-discovery), which doesn't specify any CMKs\.\)

When decrypting, the AWS Encryption SDK searches the KMS keyring for a CMK that can decrypt one of the encrypted data keys\. Specifically, the AWS Encryption SDK uses the following pattern for each encrypted data key in an encrypted message\.
+ The AWS Encryption SDK parses the metadata of the encrypted data key\. It gets the key ARN of the CMK that encrypted the data key\.
+ The AWS Encryption SDK searches the decryption keyring for a CMK with a matching key ARN\.
+ If it finds a CMK with a matching key ARN in the keyring, the AWS Encryption SDK asks AWS KMS to decrypt the encrypted data key\.
+ Otherwise, it skips to the next encrypted data key, if any\. 

The AWS Encryption SDK never attempts to decrypt an encrypted data key unless the key ARN of the CMK that encrypted that data key is included in the decryption keyring\. If the decryption keyring doesn't include the ARNs of any of the CMKs that encrypted any of the data keys, the AWS Encryption SDK fails the decrypt call without ever calling AWS KMS\.

A decrypt call with a KMS keyring succeeds when at least one CMK in the decryption keyring can decrypt one of the encrypted data keys in the encrypted message\. Also, the caller must have `kms:Decrypt` permission on that CMK\. This behavior enables you to encrypt data under multiple CMKs in different AWS Regions and accounts, but provide a more limited decryption keyring tailored to a particular account, Region, user, group, or role\. 

When you specify a CMK in a decryption keyring, you must use its key ARN\. Otherwise, the CMK is not recognized\. For help finding the key ARN, see [Finding the Key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\. 

**Note**  
If you reuse an encryption keyring for decrypting, be sure that the CMKs in the keyring are identified by their key ARNs\.

For example, the following KMS keyring includes only the additional key that was used in the encryption keyring\. You can use this keyring to decrypt a message that was encrypted under both the generator key and the additional key, provided that you have permission to use the additional key to decrypt data\.

------
#### [ C ]

```
const char * additional_key = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"    

struct aws_cryptosdk_keyring *kms_decrypt_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(additional_key);
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })
const additionalKey = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321'

const keyring = new KmsKeyringBrowser({ clientProvider, keyIds: [additionalKey] })
```

------
#### [ JavaScript Node\.js ]

```
const additionalKey = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321'

const keyring = new KmsKeyringNode({ keyIds: [additionalKey] })
```

------

You can also use a KMS keyring that specifies a generator key for decrypting, such as the following one\. When decrypting, the AWS Encryption SDK ignores the distinction between generator keys and additional keys\. It can use any of the specified CMKs to decrypt an encrypted data key\. The call to AWS KMS succeeds only when the caller has permission to use that CMK to decrypt data\.

------
#### [ C ]

```
struct aws_cryptosdk_keyring *kms_decrypt_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(generator_key, {additional_key, other_cmk});
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })

const keyring = new KmsKeyringBrowser({
  clientProvider, 
  generatorKeyId, 
  keyIds: [additionalKey, otherCmk]
})
```

------
#### [ JavaScript Node\.js ]

```
const keyring = new KmsKeyringNode({
  generatorKeyId,
  keyIds: [additionalKey, otherCmk]
})
```

------

Unlike an encryption keyring that uses all of the specified CMKs, you can decrypt an encrypted message using a decryption keyring that includes CMKs that are unrelated to the encrypted message, and CMKs that the caller doesn't have permission to use\. If a decrypt call to AWS KMS fails, such as when the caller doesn't have the required permission, the AWS Encryption SDK just skips to the next encrypted data key\. 

### Using a KMS Discovery Keyring<a name="kms-keyring-discovery"></a>

Typically, when decrypting, you provide a KMS keyring that limits the CMKs that the AWS Encryption SDK can use to those that you specify\. However, you can also create a *KMS Discovery keyring*, that is, a KMS keyring that doesn't specify any CMKs\. It allows the AWS Encryption SDK to ask AWS KMS to decrypt any encrypted data key in an encrypted message by using the CMK encrypted it, regardless of who owns or has access to that CMK\. The call succeeds only if the caller has `kms:Decrypt` permission on the CMK\.

The following code instantiates a KMS Discovery keyring\.

------
#### [ C ]

For a complete example, see: [ kms\_discovery\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/kms_discovery.cpp)\.

```
struct kms_discovery_keyring = Aws::Cryptosdk::KmsKeyring::Builder().BuildDiscovery();
```

------
#### [ JavaScript Browser ]

In JavaScript, you must explicitly specify the discovery property\.

```
const clientProvider = getClient(KMS, { credentials })
                            
const keyring = new KmsKeyringBrowser({discovery: true})
```

------
#### [ JavaScript Node\.js ]

In JavaScript, you must explicitly specify the discovery property\.

```
const keyring = new KmsKeyringNode({discovery: true})
```

------

When encrypting, a KMS Discovery keyring has no effect\. It doesn't return any encrypted data keys\. However, you might want to include a KMS Discovery keyring in a multi\-keyring that will be used for encrypting and decrypting\.

When decrypting with a KMS Discovery keyring, the AWS Encryption SDK calls AWS KMS to decrypt each of the encrypted data keys in the encrypted message in the order that they appear until one succeeds\. To succeed, the caller must have `kms:Decrypt` permission on at least one of the CMKs that encrypted one of the encrypted data keys\.

The AWS Encryption SDK provides a KMS Discovery keyring for convenience\. However, we recommend that you use a more limited keyring whenever possible for the following reasons\.
+ **Authenticity** – The KMS Discovery keyring can use any CMK that was used to encrypt a data key in the encrypted message, just so the caller has permission to use that CMK to decrypt\. This might not be the CMK that the caller intends to use\. For example, one of the encrypted data keys might have been encrypted under a less secure CMK that anyone can use\. 
+ **Latency and performance** – A KMS Discovery keyring might be perceptibly slower than other keyrings because the AWS Encryption SDK tries to decrypt all of the encrypted data keys, including those encrypted by CMKs in other AWS accounts and Regions, and CMKs that the caller doesn't have permission to use for decryption\. 

When you use a KMS Discovery keyring in a multi\-keyring, it has no effect on encryption\. When decrypting, by allowing the use of any CMK, a KMS Discovery keyring overrides any CMK limits that other KMS keyrings in the multi\-keyring might impose\. For example, if you combine a KMS keyring that uses a particular CMK with a KMS Discovery keyring, the resulting multi\-keyring behaves just like the KMS Discovery keyring\. It lets the AWS Encryption SDK ask AWS KMS to decrypt any encrypted data key, even if it was encrypted by a different CMK\. 

### Using a KMS Regional Discovery Keyring<a name="kms-keyring-regional"></a>

Instead of using a KMS keyring that specifies particular CMKs, or a KMS Discovery keyring that can use any CMK, you might want to use a KMS Regional Discovery keyring that includes or excludes the AWS Encryption SDK to CMKs in a particular AWS Region\.

For example, the following code creates a KMS Regional Discovery keyring that uses only CMKs in the US West \(Oregon\) Region \(us\-west\-2\)\. 

------
#### [ C ]

To view this keyring, and the `create_kms_client` method, in a working example, see [kms\_discovery\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/kms_discovery.cpp)\.

```
struct aws_cryptosdk_keyring *kms_regional_keyring = Aws::Cryptosdk::KmsKeyring::Builder()
       .WithKmsClient(create_kms_client(Aws::Region::US_WEST_2)).BuildDiscovery());
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })

const discovery = true
const clientProvider = limitRegions(['us-west-2'], getKmsClient)
const keyring = new KmsKeyringBrowser({ clientProvider, discovery })
```

------
#### [ JavaScript Node\.js ]

To view this keyring, and the `limitRegions` and function, in a working example, see [kms\_regional\_discovery\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_regional_discovery.ts)\.

```
const discovery = true
const clientProvider = limitRegions(['us-west-2'], getKmsClient)
const keyring = new KmsKeyringNode({ clientProvider, discovery })
```

------

The AWS Encryption SDK for JavaScript also exports an `excludeRegions` function for Node\.js and the browser\. This function creates a KMS Regional Discovery keyring that omits CMKs in particular regions\. The following example creates a KMS Regional Discovery keyring that can use CMKs in every AWS Region except for US East \(N\. Virginia\) \(us\-east\-1\)\. 

The AWS Encryption SDK for C does not have an analogous method, but you can implement one by creating a custom [ClientSupplier](https://github.com/aws/aws-encryption-sdk-c/blob/master/aws-encryption-sdk-cpp/include/aws/cryptosdk/cpp/kms_keyring.h#L157)\.

This example shows the code for Node\.js\.

```
const discovery = true
const clientProvider = excludeRegions(['us-east-1'], getKmsClient)
const keyring = new KmsKeyringNode({ clientProvider, discovery })
```

When encrypting, a KMS Regional Discovery keyring has no effect\. Because it doesn't specify any CMKs, it can't generate or encrypt data keys\. However, you can include a KMS Regional Discovery keyring in a multi\-keyring that will be used for encrypting and decrypting\. 

When decrypting with a KMS Regional Discovery keyring, the AWS Encryption SDK can ask AWS KMS to decrypt any encrypted data key that was encrypted under a CMK in the specified AWS Region\. To succeed, the caller must have `kms:Decrypt` permission on at least one of the CMKs in the specified AWS Region that encrypted one of the data keys in the encrypted message\.

In a multi\-keyring, because it allows the use of any CMK in the AWS Region, a KMS Regional Discovery keyring can override CMK limits that other KMS keyrings in the multi\-keyring might impose\. For example, if you combine a KMS keyring that lets you use a particular CMK in the EU \(London\) Region and a KMS Regional Discovery keyring for the EU \(London\) Region, the resulting multi\-keyring behaves just like the KMS Regional Discovery keyring alone\. It lets the AWS Encryption SDK ask AWS KMS to decrypt any encrypted data key that was encrypted by any CMK in the EU \(London\) Region\. 

## Raw AES Keyrings<a name="use-raw-aes-keyring"></a>

The Raw AES keyring uses the AES\-GCM algorithm and a wrapping key that you specify as a byte array to encrypt data keys\. You can specify only one wrapping key in each Raw AES keyring, but you can include multiple Raw AES keyrings in each [multi\-keyring](#use-multi-keyring)\. 

The Raw AES keyring is equivalent to and interoperates with the [JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) in the AWS Encryption SDK for Java and the [RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) in the AWS Encryption SDK for Python when they are used with symmetric encryption keys\. You can encrypt data with one implementation and decrypt the data with any other implementation using the same wrapping key\.

Use a Raw AES keyring when you need to provide the wrapping key and encrypt the data keys locally, or you need to write an application that is compatible with the AWS Encryption SDK for Java or the AWS Encryption SDK for Python\. Whenever possible, we recommend using a hardware security module \(HSM\) or a service, such as AWS KMS, that doesn't expose wrapping keys and encrypts data keys within a secure boundary\.

To identify the wrapping key, the Raw AES keyring uses a namespace and name that you provide\. These are equivalent to the `Provider ID` and `Key ID` fields in the AWS Encryption SDK for Java and the AWS Encryption SDK for Python\. These values are not secret\. They appear in plaintext in the header of the [encrypted message](concepts.md#message) that the AWS Encryption SDK returns\. However, they are critical\. You must use the same namespace and name for a key in your decryption keyring that you used for that key in your encryption keyring\. If the namespace and name are not an exact, case\-sensitive match, the AWS Encryption SDK will not recognize that the wrapping keys are the same, even if the bytes are identical, and it will not be able to decrypt the encrypted data keys\.

For an example of how to use a Raw AES keyring, see:
+ C: [raw\_aes\_keyring\.c](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/raw_aes_keyring.c)
+ JavaScript Node\.js: [aes\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/aes_simple.ts)
+ JavaScript Browser: [aes\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/aes_simple.ts)

## Raw RSA Keyrings<a name="use-raw-rsa-keyring"></a>

The Raw RSA keyring performs asymmetric encryption and decryption of data keys in local memory with an RSA public and private keys that you specify\. The encryption function encrypts the data key under the RSA public key\. The decryption function decrypts the data key using the private key\. You can select from among the several [RSA padding modes](https://github.com/aws/aws-encryption-sdk-c/blob/master/include/aws/cryptosdk/cipher.h)\.

A Raw RSA keyring that encrypts and decrypts must include an asymmetric public key and private key pair\. But you can encrypt data with a Raw RSA keyring that has only a public key, and you can decrypt data with a Raw RSA keyring that has only a private key\. And you can include any Raw RSA keyring in a [multi\-keyring](#use-multi-keyring)\. The Raw RSA keyring is equivalent to and interoperates with the [JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) in the AWS Encryption SDK for Java and the [RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) in the AWS Encryption SDK for Python when they are used with asymmetric encryption keys\. You can encrypt data with one implementation and decrypt the data with any other implementation using the same wrapping key\.

Use a Raw RSA keyring when you want to use an asymmetric key pair and provide the wrapping key and unwrapping keys, or you need to be compatible with the AWS Encryption SDK for another programming language\. 

To identify the key pair, the Raw RSA keyring uses a namespace and name that you provide\. These values are not secret\. They appear in plaintext in the header of the [encrypted message](concepts.md#message) that the AWS Encryption SDK returns\. However, they are critical\. If you use the same key pair in an encryption keyring and a decryption keyring, be sure to use the same namespace and name for the key pair in both keyrings\. If the namespace and name are not an exact, case\-sensitive match, the AWS Encryption SDK will not recognize that the asymmetric keys are a pair, and it will not be able to decrypt the encrypted data keys\.

When constructing a Raw RSA keyring in the AWS Encryption SDK for C, be sure to provide the *contents* of the PEM file that includes each key as a null\-terminated C\-string, not as a path or file name\. When constructing a Raw RSA keyring in JavaScript, be aware of [potential incompatibility](javascript.md#javascript-compatibility) with other language implementations\.

For an example of how to use a Raw RSA keyring, see:
+ C: [raw\_rsa\_keyring\.c](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/raw_rsa_keyring.c)
+ JavaScript Node\.js: [rsa\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/rsa_simple.ts)
+ JavaScript Browser: [rsa\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/rsa_simple.ts)

## Multi\-Keyrings<a name="use-multi-keyring"></a>

You can combine keyrings into a multi\-keyring\. A *multi\-keyring* is a keyring that consists of one or more individual keyrings of the same or a different type\. The effect is like using several keyrings in a series\. When you use a multi\-keyring to encrypt data, any of the wrapping keys in any of its keyrings can decrypt that data\.

When you create a multi\-keyring to encrypt data, you designate one of the keyrings as the *generator keyring*\. All other keyrings are known as *child keyrings*\. The generator keyring generates and encrypts the plaintext data key\. Then, all of the wrapping keys in all of the child keyrings encrypt the same plaintext data key\. The multi\-keyring returns the plaintext key and one encrypted data key for each wrapping key in the multi\-keyring\. If you create a multi\-keyring with no generator keyring, you can use it to decrypt data, but not to encrypt\. If the generator keyring is a [KMS keyring](#use-kms-keyring), the generator key in the KMS keyring generates and encrypts the plaintext key\. Then, all additional CMKs in the KMS keyring, and all wrapping keys in all child keyrings in the multi\-keyring, encrypt the same plaintext key\. 

When decrypting, the AWS Encryption SDK uses the keyrings to try to decrypt one of the encrypted data keys\. The keyrings are called in the order that they are specified in the multi\-keyring\. Processing stops as soon as any key in any keyring can decrypt an encrypted data key\. 

To see a working example of a multi\-keyring, see:
+ C: [multi\_keyring\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/multi_keyring.cpp)[]()
+ JavaScript Node\.js: [multi\_keyring\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/multi_keyring.ts)
+ JavaScript Browser: [multi\_keyring\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/multi_keyring.ts)

To create a multi\-keyring, first instantiate the child keyrings\. In this example, we use a KMS keyring and a Raw AES keyring, but you can combine any supported keyrings in a multi\-keyring\.

------
#### [ C ]

```
// Define a KMS keyring. For details, see [string\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/string.cpp).
struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(example_CMK);

// Define a Raw AES keyring. For details, see [raw\_aes\_keyring\.c](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/raw_aes_keyring.c).
struct aws_cryptosdk_keyring *aes_keyring = aws_cryptosdk_raw_aes_keyring_new(
        alloc, wrapping_key_namespace, wrapping_key_name, wrapping_key, AWS_CRYPTOSDK_AES256);
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })

// Define a KMS keyring. For details, see [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/kms_simple.ts). 
const kmsKeyring = new KmsKeyringBrowser({ generatorKeyId: exampleCmk })

// Define a Raw AES keyring. For details, see [aes\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/aes_simple.ts).
const aesKeyring = new RawAesKeyringWebCrypto({ keyName, keyNamespace, wrappingSuite, masterKey })
```

------
#### [ JavaScript Node\.js ]

```
// Define a KMS keyring. For details, see [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_simple.ts). 
const kmsKeyring = new KmsKeyringNode({ generatorKeyId: exampleCmk })

// Define a Raw AES keyring. For details, see [raw\_aes\_keyring\_node\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/raw-aes-keyring-node/src/raw_aes_keyring_node.ts).
const aesKeyring = new RawAesKeyringNode({ keyName, keyNamespace, wrappingSuite, unencryptedMasterKey })
```

------

Next, create the multi\-keyring and specify its generator keyring, if any\. In this example, we create a multi\-keyring in which the KMS keyring is the generator keyring and the AES keyring is the child keyring\.

------
#### [ C ]

In the multi\-keyring constructor in C, you specify only its generator keyring\. 

```
struct aws_cryptosdk_keyring *multi_keyring = aws_cryptosdk_multi_keyring_new(alloc, kms_keyring);
```

To add a child keyring to your multi\-keyring, use the `aws_cryptosdk_multi_keyring_add_child` method\. You need to call the method once for each child keyring that you add\. 

```
/* Add the Raw AES keyring (C only)
int aws_cryptosdk_multi_keyring_add_child(struct aws_cryptosdk_keyring *multi, struct aws_cryptosdk_keyring *child);
```

------
#### [ JavaScript Browser ]

JavaScript multi\-keyrings are immutable\. The JavaScript multi\-keyring constructor lets you specify the generator keyring and multiple child keyrings\. 

```
const clientProvider = getClient(KMS, { credentials })

const multiKeyring = new MultiKeyringWebCrypto(generator: kmsKeyring, children: [aesKeyring]);
```

------
#### [ JavaScript Node\.js ]

JavaScript multi\-keyrings are immutable\. The JavaScript multi\-keyring constructor lets you specify the generator keyring and multiple child keyrings\. 

```
const multiKeyring = new MultiKeyringNode(generator: kmsKeyring, children: [aesKeyring]);
```

------

Now, you can use the multi\-keyring to encrypt and decrypt data\. 

For a complete example of how to build and use a multi\-keyring in C, see [multi\_keyring\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/multi_keyring.cpp)\.