# AWS Encryption SDK Message Format Reference<a name="message-format"></a>


|  | 
| --- |
|  The information on this page is a reference for building your own encryption library that is compatible with the AWS Encryption SDK\. If you are not building your own compatible encryption library, you likely do not need this information\. To use the AWS Encryption SDK in one of the supported programming languages, see [Programming Languages](programming-languages.md)\.  | 

The encryption operations in the AWS Encryption SDK return a single data structure or *message* that contains the encrypted data \(ciphertext\) and all encrypted data keys\. To understand this data structure, or to build libraries that read and write it, you need to understand the message format\.

The message format consists of at least two parts: a *header* and a *body*\. In some cases, the message format consists of a third part, a *footer*\. The message format defines an ordered sequence of bytes in network byte order, also called big\-endian format\. The message format begins with the header, followed by the body, followed by the footer \(when there is one\)\.

**Topics**
+ [Header Structure](#header-structure)
+ [Body Structure](#body-structure)
+ [Footer Structure](#footer-structure)

## Header Structure<a name="header-structure"></a>

The message header contains the encrypted data key and information about how the message body is formed\. The following table describes the fields that form the header\. The bytes are appended in the order shown\.


**Header Structure**  

| Field | Length \(bytes\) | 
| --- | --- | 
| [Version](#header-version) | 1 | 
| [Type](#header-type) | 1 | 
| [Algorithm ID](#header-algorithm-id) | 2 | 
| [Message ID](#header-message-id) | 16 | 
| [AAD Length](#header-aad-length) | 2When the [encryption context](concepts.md#encryption-context) is empty, the AAD Length is 0\. | 
| [AAD](#header-aad) | Variable\. Equal to the value specified in the previous 2 bytes \(AAD Length\)\. When the [encryption context](concepts.md#encryption-context) is empty, there is no AAD field in the header\. | 
| [Encrypted Data Key Count](#header-data-key-count) | 2 | 
| [Encrypted Data Key\(s\)](#header-data-keys) | Variable\. Determined by the number of encrypted data keys and the length of each\. | 
| [Content Type](#header-content-type) | 1 | 
| [Reserved](#header-reserved) | 4 | 
| [IV Length](#header-iv-length) | 1 | 
| [Frame Length](#header-frame-length) | 4 | 
| [Header Authentication](#header-authentication) | Variable\. Determined by the [algorithm](algorithms-reference.md) that generated the message\. | 

**Version**  <a name="header-version"></a>
The version of this message format\. The current version is 1\.0, encoded as the byte `01` in hexadecimal notation\.

**Type**  <a name="header-type"></a>
The type of this message format\. The type indicates the kind of structure\. The only supported type is described as *customer authenticated encrypted data*\. Its type value is 128, encoded as byte `80` in hexadecimal notation\.

**Algorithm ID**  <a name="header-algorithm-id"></a>
An identifier for the algorithm used\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer\. For more information about the algorithms, see [AWS Encryption SDK Algorithms Reference](algorithms-reference.md)\.

**Message ID**  <a name="header-message-id"></a>
A randomly generated 128\-bit value that identifies the message\. The Message ID:  
+ Uniquely identifies the encrypted message\.
+ Weakly binds the message header to the message body\.
+ Provides a mechanism to securely reuse a data key with multiple encrypted messages\.
+ Protects against accidental reuse of a data key or the wearing out of keys in the AWS Encryption SDK\.

**AAD Length**  <a name="header-aad-length"></a>
The length of the additional authenticated data \(AAD\)\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer that specifies the number of bytes that contain the AAD\.  
When the [encryption context](concepts.md#encryption-context) is empty, the AAD Length is 0\.

**AAD**  <a name="header-aad"></a>
The additional authenticated data\. The AAD is an encoding of the [encryption context](concepts.md#encryption-context), an array of key\-value pairs where each key and value is a string of UTF\-8 encoded characters\. The encryption context is converted to a sequence of bytes and used for the AAD value\. When the encryption context is empty, there is no AAD field in the header\.  
When the [algorithms with signing](algorithms-reference.md) are used, the encryption context must contain the key\-value pair `{'aws-crypto-public-key', Qtxt}`\. Qtxt represents the elliptic curve point Q compressed according to [SEC 1 version 2\.0](http://www.secg.org/sec1-v2.pdf) and then base64\-encoded\. The encryption context can contain additional values, but the maximum length of the constructed AAD is 2^16 \- 1 bytes\.  
The following table describes the fields that form the AAD\. Key\-value pairs are sorted, by key, in ascending order according to UTF\-8 character code\. The bytes are appended in the order shown\.    
**AAD Structure**    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/message-format.html)  
**Key\-Value Pair Count**  <a name="aad-count"></a>
The number of key\-value pairs in the AAD\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer that specifies the number of key\-value pairs in the AAD\. The maximum number of key\-value pairs in the AAD is 2^16 \- 1\.  
When there is no encryption context or the encryption context is empty, this field is not present in the AAD structure\.  
**Key Length**  <a name="aad-key-length"></a>
The length of the key for the key\-value pair\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer that specifies the number of bytes that contain the key\.  
**Key**  <a name="aad-key"></a>
The key for the key\-value pair\. It is a sequence of UTF\-8 encoded bytes\.  
**Value Length**  <a name="aad-value-length"></a>
The length of the value for the key\-value pair\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer that specifies the number of bytes that contain the value\.  
**Value**  <a name="aad-value"></a>
The value for the key\-value pair\. It is a sequence of UTF\-8 encoded bytes\.

**Encrypted Data Key Count**  <a name="header-data-key-count"></a>
The number of encrypted data keys\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer that specifies the number of encrypted data keys\.

**Encrypted Data Key\(s\)**  <a name="header-data-keys"></a>
A sequence of encrypted data keys\. The length of the sequence is determined by the number of encrypted data keys and the length of each\. The sequence contains at least one encrypted data key\.  
The following table describes the fields that form each encrypted data key\. The bytes are appended in the order shown\.    
**Encrypted Data Key Structure**    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/message-format.html)  
**Key Provider ID Length**  <a name="data-key-provider-id-length"></a>
The length of the key provider identifier\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer that specifies the number of bytes that contain the key provider ID\.  
**Key Provider ID**  <a name="data-key-provider-id"></a>
The key provider identifier\. It is used to indicate the provider of the encrypted data key and intended to be extensible\.  
**Key Provider Information Length**  <a name="data-key-provider-info-length"></a>
The length of the key provider information\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer that specifies the number of bytes that contain the key provider information\.  
**Key Provider Information**  <a name="data-key-provider-info"></a>
The key provider information\. It is determined by the key provider\.  
When AWS KMS is the master key provider or you are using a KMS keyring, this value contains the Amazon Resource Name \(ARN\) of the AWS KMS customer master key \(CMK\)\.  
**Encrypted Data Key Length**  <a name="data-key-length"></a>
The length of the encrypted data key\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer that specifies the number of bytes that contain the encrypted data key\.  
**Encrypted Data Key**  <a name="data-key"></a>
The encrypted data key\. It is the data encryption key encrypted by the key provider\.

**Content Type**  <a name="header-content-type"></a>
The type of encrypted content, either nonframed or framed\.  
Non\-framed content is not broken into parts; it is a single encrypted blob\. Non\-framed content is type 1, encoded as the byte `01` in hexadecimal notation\.  
Framed content is broken into equal\-length parts; each part is encrypted separately\. Framed content is type 2, encoded as the byte `02` in hexadecimal notation\.

**Reserved**  <a name="header-reserved"></a>
A reserved sequence of 4 bytes\. This value must be 0\. It is encoded as the bytes `00 00 00 00` in hexadecimal notation \(that is, a 4\-byte sequence of a 32\-bit integer value equal to 0\)\.

**IV Length**  <a name="header-iv-length"></a>
The length of the initialization vector \(IV\)\. It is a 1\-byte value interpreted as an 8\-bit unsigned integer that specifies the number of bytes that contain the IV\. This value is determined by the IV bytes value of the [algorithm](algorithms-reference.md) that generated the message\.

**Frame Length**  <a name="header-frame-length"></a>
The length of each frame of framed content\. It is a 4\-byte value interpreted as a 32\-bit unsigned integer that specifies the number of bytes that form each frame\. When the content is nonframed—that is, when the value of the content type field is 1—this value must be 0\.

**Header Authentication**  <a name="header-authentication"></a>
The header authentication is determined by the [algorithm](algorithms-reference.md) that generated the message\. The header authentication is calculated over the entire header\. It consists of an IV and an authentication tag\. The bytes are appended in the order shown\.    
**Header Authentication Structure**    
[\[See the AWS documentation website for more details\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/message-format.html)  
**IV**  <a name="header-authentication-iv"></a>
The initialization vector \(IV\) used to calculate the header authentication tag\.  
**Authentication Tag**  <a name="header-authentication-tag"></a>
The authentication value for the header\. It is used to authenticate the entire contents of the header\.

## Body Structure<a name="body-structure"></a>

The message body contains the encrypted data, called the *ciphertext*\. The structure of the body depends on the content type \(nonframed or framed\)\. The following sections describe the format of the message body for each content type\.

**Topics**
+ [Non\-Framed Data](#body-no-framing)
+ [Framed Data](#body-framing)

### Non\-Framed Data<a name="body-no-framing"></a>

Non\-framed data is encrypted in a single blob with a unique IV and [body AAD](body-aad-reference.md)\. The following table describes the fields that form nonframed data\. The bytes are appended in the order shown\.


**Non\-Framed Body Structure**  

| Field | Length, in bytes | 
| --- | --- | 
| [IV](#body-unframed-iv) | Variable\. Equal to the value specified in the [IV Length](#header-iv-length) byte of the header\. | 
| [Encrypted Content Length](#body-unframed-content-length) | 8 | 
| [Encrypted Content](#body-unframed-content) | Variable\. Equal to the value specified in the previous 8 bytes \(Encrypted Content Length\)\. | 
| [Authentication Tag](#body-unframed-tag) | Variable\. Determined by the [algorithm implementation](algorithms-reference.md) used\. | 

**IV**  <a name="body-unframed-iv"></a>
The initialization vector \(IV\) to use with the [encryption algorithm](algorithms-reference.md)\.

**Encrypted Content Length**  <a name="body-unframed-content-length"></a>
The length of the encrypted content, or *ciphertext*\. It is an 8\-byte value interpreted as a 64\-bit unsigned integer that specifies the number of bytes that contain the encrypted content\.  
Technically, the maximum allowed value is 2^63 \- 1, or 8 exbibytes \(8 EiB\)\. However, in practice the maximum value is 2^36 \- 32, or 64 gibibytes \(64 GiB\), due to restrictions imposed by the [implemented algorithms](algorithms-reference.md)\.  
The Java implementation of this SDK further restricts this value to 2^31 \- 1, or 2 gibibytes \(2 GiB\), due to restrictions in the language\.

**Encrypted Content**  <a name="body-unframed-content"></a>
The encrypted content \(ciphertext\) as returned by the [encryption algorithm](algorithms-reference.md)\.

**Authentication Tag**  <a name="body-unframed-tag"></a>
The authentication value for the body\. It is used to authenticate the message body\.

### Framed Data<a name="body-framing"></a>

Framed data is divided into equal\-length parts, except for the last part\. Each frame is encrypted separately with a unique IV and [body AAD](body-aad-reference.md)\.

There are two kinds of frames: regular and final\. A final frame is always used\. When the length of the data is an exact multiple of the frame length, the final frame contains no data—that is, it has a content length of 0\. When the length of the data is less than the frame length, only a final frame is written\.

The following tables describe the fields that form the frames\. The bytes are appended in the order shown\.


**Framed Body Structure, Regular Frame**  

| Field | Length, in bytes | 
| --- | --- | 
| [Sequence Number](#body-framed-regular-sequence-number) | 4 | 
| [IV](#body-framed-regular-iv) | Variable\. Equal to the value specified in the [IV Length](#header-iv-length) byte of the header\. | 
| [Encrypted Content](#body-framed-regular-content) | Variable\. Equal to the value specified in the [Frame Length](#header-frame-length) of the header\. | 
| [Authentication Tag](#body-framed-regular-tag) | Variable\. Determined by the algorithm used, as specified in the [Algorithm ID](#header-algorithm-id) of the header\. | 

**Sequence Number**  <a name="body-framed-regular-sequence-number"></a>
The frame sequence number\. It is an incremental counter number for the frame\. It is a 4\-byte value interpreted as a 32\-bit unsigned integer\.  
Framed data must start at sequence number 1\. Subsequent frames must be in order and must contain an increment of 1 of the previous frame\. Otherwise, the decryption process stops and reports an error\.

**IV**  <a name="body-framed-regular-iv"></a>
The initialization vector \(IV\) for the frame\. The SDK uses a deterministic method to construct a different IV for each frame in the message\. Its length is specified by the [algorithm suite](algorithms-reference.md) used\.

**Encrypted Content**  <a name="body-framed-regular-content"></a>
The encrypted content \(ciphertext\) for the frame, as returned by the [encryption algorithm](algorithms-reference.md)\.

**Authentication Tag**  <a name="body-framed-regular-tag"></a>
The authentication value for the frame\. It is used to authenticate the entire frame\.


**Framed Body Structure, Final Frame**  

| Field | Length, in bytes | 
| --- | --- | 
| [Sequence Number End](#body-framed-final-sequence-number-end) | 4 | 
| [Sequence Number](#body-framed-final-sequence-number) | 4 | 
| [IV](#body-framed-final-iv) | Variable\. Equal to the value specified in the [IV Length](#header-iv-length) byte of the header\. | 
| [Encrypted Content Length](#body-framed-final-content-length) | 4 | 
| [Encrypted Content](#body-framed-final-content) | Variable\. Equal to the value specified in the previous 4 bytes \(Encrypted Content Length\)\. | 
| [Authentication Tag](#body-framed-final-tag) | Variable\. Determined by the algorithm used, as specified in the [Algorithm ID](#header-algorithm-id) of the header\. | 

**Sequence Number End**  <a name="body-framed-final-sequence-number-end"></a>
An indicator for the final frame\. The value is encoded as the 4 bytes `FF FF FF FF` in hexadecimal notation\.

**Sequence Number**  <a name="body-framed-final-sequence-number"></a>
The frame sequence number\. It is an incremental counter number for the frame\. It is a 4\-byte value interpreted as a 32\-bit unsigned integer\.  
Framed data must start at sequence number 1\. Subsequent frames must be in order and must contain an increment of 1 of the previous frame\. Otherwise, the decryption process stops and reports an error\.

**IV**  <a name="body-framed-final-iv"></a>
The initialization vector \(IV\) for the frame\. The SDK uses a deterministic method to construct a different IV for each frame in the message\. The length of the IV length is specified by the [algorithm suite](algorithms-reference.md)\.

**Encrypted Content Length**  <a name="body-framed-final-content-length"></a>
The length of the encrypted content\. It is a 4\-byte value interpreted as a 32\-bit unsigned integer that specifies the number of bytes that contain the encrypted content for the frame\.

**Encrypted Content**  <a name="body-framed-final-content"></a>
The encrypted content \(ciphertext\) for the frame, as returned by the [encryption algorithm](algorithms-reference.md)\.

**Authentication Tag**  <a name="body-framed-final-tag"></a>
The authentication value for the frame\. It is used to authenticate the entire frame\.

## Footer Structure<a name="footer-structure"></a>

When the [algorithms with signing](algorithms-reference.md) are used, the message format contains a footer\. The message footer contains a signature calculated over the message header and body\. The following table describes the fields that form the footer\. The bytes are appended in the order shown\.


**Footer Structure**  

| Field | Length, in bytes | 
| --- | --- | 
| [Signature Length](#footer-signature-length) | 2 | 
| [Signature](#footer-signature) | Variable\. Equal to the value specified in the previous 2 bytes \(Signature Length\)\. | 

**Signature Length**  <a name="footer-signature-length"></a>
The length of the signature\. It is a 2\-byte value interpreted as a 16\-bit unsigned integer that specifies the number of bytes that contain the signature\.

**Signature**  <a name="footer-signature"></a>
The signature\. It is used to authenticate the header and body of the message\.