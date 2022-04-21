# How Amazon Elastic Transcoder uses AWS KMS<a name="services-et"></a>

You can use Amazon Elastic Transcoder to convert media files stored in an Amazon S3 bucket into formats required by consumer playback devices\. Both input and output files can be encrypted and decrypted\. The following sections discuss how AWS KMS is used for both processes\.

**Topics**
+ [Encrypting the input file](#et-encrypt-input)
+ [Decrypting the input file](#et-decrypt-input)
+ [Encrypting the output file](#et-output)
+ [HLS content protection](#et-hls)
+ [Elastic Transcoder encryption context](#et-encryption-context)

## Encrypting the input file<a name="et-encrypt-input"></a>

Before you can use Elastic Transcoder, you must [create an Amazon S3 bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-bucket.html) and upload your media file into it\. You can encrypt the file before uploading by using AES client\-side encryption or after uploading by using Amazon S3 server\-side encryption\.

If you choose client\-side encryption using AES, you are responsible for encrypting the file before uploading it to Amazon S3, and you must provide Elastic Transcoder access to the encryption key\. You do this by using a [symmetric](concepts.md#symmetric-cmks) AWS KMS [AWS KMS key](concepts.md#kms_keys) to protect the AES encryption key you used to encrypt the media file\.

If you choose server\-side encryption, you allow Amazon S3 to encrypt and decrypt all files on your behalf\. You can configure Amazon S3 to use one of three different types of encryption keys to protect the unique data key that encrypts your file:
+ An Amazon S3 key, an encryption key that Amazon S3 owns and manages\. It is not part of your AWS account\.
+ The [AWS managed key](concepts.md#aws-managed-cmk) for Amazon S3, a KMS key that is part of your account, but is created and managed by AWS
+ Any [symmetric](concepts.md#symmetric-cmks) [customer managed key](concepts.md#customer-cmk) that you create by using AWS KMS

**Important**  
For both client\-side and server\-side encryption, Elastic Transcoder supports only [symmetric KMS keys](concepts.md#symmetric-cmks)\. You cannot use an [asymmetric KMS key](symmetric-asymmetric.md#asymmetric-cmks) to encrypt your Elastic Transcoder files\. For help determining whether a KMS key is symmetric or asymmetric, see [Identifying asymmetric KMS keys](find-symm-asymm.md)\.

You can enable encryption and specify a key by using the Amazon S3 console or the appropriate Amazon S3 APIs\. For more information about how Amazon S3 performs encryption, see [Protecting data using server\-side encryption with KMS keys \(SSE\-KMS\)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html) in the *Amazon Simple Storage Service User Guide*\.

When you protect your input file by using the AWS managed key for Amazon S3 in your account or a customer managed key, Amazon S3 and AWS KMS interact in the following manner:

1. Amazon S3 requests a plaintext data key and a copy of the data key encrypted under the specified KMS key\.

1. AWS KMS creates a data key, encrypts it with the specified KMS key, and then sends both the plaintext data key and the encrypted data key to Amazon S3\.

1. Amazon S3 uses the plaintext data key to encrypt the media file and then stores the file in the specified Amazon S3 bucket\.

1. Amazon S3 stores the encrypted data key alongside of the encrypted media file\.

## Decrypting the input file<a name="et-decrypt-input"></a>

If you choose Amazon S3 server\-side encryption to encrypt the input file, Elastic Transcoder does not decrypt the file\. Instead, Elastic Transcoder relies on Amazon S3 to perform decryption depending on the [settings you specify when you create a job](https://docs.aws.amazon.com/elastictranscoder/latest/developerguide/job-settings.html) and a pipeline\. 

The following combination of settings are available\.


****  

| Encryption mode | AWS KMS key | Meaning | 
| --- | --- | --- | 
| S3 | Default | Amazon S3 creates and manages the keys used to encrypt and decrypt the media file\. The process is opaque to the user\. | 
| S3\-AWS\-KMS | Default | Amazon S3 uses a data key encrypted by the default AWS managed key for Amazon S3 in your account to encrypt the media file\. | 
| S3\-AWS\-KMS | Custom \(with ARN\) | Amazon S3 uses a data key encrypted by the specified customer managed key to encrypt the media file\. | 

When `S3-AWS-KMS` is specified, Amazon S3 and AWS KMS work together in the following manner to perform the decryption\.

1. Amazon S3 sends the encrypted data key to AWS KMS\.

1. AWS KMS decrypts the data key by using the appropriate KMS key, and then sends the plaintext data key back to Amazon S3\.

1. Amazon S3 uses the plaintext data key to decrypt the ciphertext\.

If you choose client\-side encryption using an AES key, Elastic Transcoder retrieves the encrypted file from the Amazon S3 bucket and decrypts it\. Elastic Transcoder uses the KMS key you specified when you created the pipeline to decrypt the AES key and then uses the AES key to decrypt the media file\.

## Encrypting the output file<a name="et-output"></a>

Elastic Transcoder encrypts the output file depending on how you specify the encryption settings when you create a job and a pipeline\. The following options are available\.


****  

| Encryption mode | AWS KMS key | Meaning | 
| --- | --- | --- | 
| S3 | Default | Amazon S3 creates and manages the keys used to encrypt the output file\. | 
| S3\-AWS\-KMS | Default | Amazon S3 uses a data key created by AWS KMS and encrypted by the AWS managed key for Amazon S3 in your account\. | 
| S3\-AWS\-KMS | Custom \(with ARN\) | Amazon S3 uses a data key encrypted by using the customer managed key specified by the ARN to encrypt the media file\. | 
| AES\- | Default | Elastic Transcoder uses the AWS managed key for Amazon S3 in your account to decrypt the specified AES key you provide and uses that key to encrypt the output file\. | 
| AES\- | Custom \(with ARN\) | Elastic Transcoder uses the customer managed key specified by the ARN to decrypt the specified AES key you provide and uses that key to encrypt the output file\. | 

When you specify that the AWS managed key for Amazon S3 in your account or a customer managed key is used to encrypt the output file, Amazon S3 and AWS KMS interact in the following manner:

1. Amazon S3 requests a plaintext data key and a copy of the data key encrypted under the specified KMS key\.

1. AWS KMS creates a data key, encrypts it under the KMS key, and sends both the plaintext data key and the encrypted data key to Amazon S3\.

1. Amazon S3 encrypts the media using the data key and stores it in the specified Amazon S3 bucket\.

1. Amazon S3 stores the encrypted data key alongside the encrypted media file\.

When you specify that your provided AES key be used to encrypt the output file, the AES key must be encrypted using a KMS key in AWS KMS\. Elastic Transcoder, AWS KMS, and you interact in the following manner:

1. You encrypt your AES key by calling the [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation in the AWS KMS API\. AWS KMS encrypts the key by using the specified KMS key\. You specify which KMS key to use when you are creating the pipeline\.

1. You specify the file containing the encrypted AES key when you create the Elastic Transcoder job\.

1. Elastic Transcoder decrypts the key by calling the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation in the AWS KMS API, passing the encrypted key as ciphertext\.

1. Elastic Transcoder uses the decrypted AES key to encrypt the output media file and then deletes the decrypted AES key from memory\. Only the encrypted copy you originally defined in the job is saved to disk\.

1. You can download the encrypted output file and decrypt it locally by using the original AES key that you defined\.

**Important**  
AWS never stores your private encryption keys\. Therefore, it is important that you manage your keys safely and securely\. If you lose them, you won't be able to decrypt your data\.

## HLS content protection<a name="et-hls"></a>

HTTP Live Streaming \(HLS\) is an adaptive streaming protocol\. Elastic Transcoder supports HLS by breaking your input file into smaller individual files called *media segments*\. A set of corresponding individual media segments contain the same material encoded at different bit rates, thereby enabling the player to select the stream that best fits the available bandwidth\. Elastic Transcoder also creates playlists that contain metadata for the various segments that are available to be streamed\.

When you enable *HLS content protection*, each media segment is encrypted using a 128\-bit AES encryption key\. When the content is viewed, during the playback process, the player downloads the key and decrypts the media segments\.

Two types of keys are used: an KMS key and a data key\. You must create a KMS key to use to encrypt and decrypt the data key\. Elastic Transcoder uses the data key to encrypt and decrypt media segments\. The data key must be AES\-128\. All variations and segments of the same content are encrypted using the same data key\. You can provide a data key or have Elastic Transcoder create it for you\.

The KMS key can be used to encrypt the data key at the following points:
+ If you provide your own data key, you must encrypt it before passing it to Elastic Transcoder\.
+ If you request that Elastic Transcoder generate the data key, then Elastic Transcoder encrypts the data key for you\.

The KMS key can be used to decrypt the data key at the following points:
+ Elastic Transcoder decrypts your provided data key when it needs to use the data key to encrypt the output file or decrypt the input file\.
+ You decrypt a data key generated by Elastic Transcoder and use it to decrypt output files\.

For more information, see [HLS Content Protection](https://docs.aws.amazon.com/elastictranscoder/latest/developerguide/content-protection.html) in the *Amazon Elastic Transcoder Developer Guide*\.

## Elastic Transcoder encryption context<a name="et-encryption-context"></a>

An [encryption context](concepts.md#encrypt_context) is a set of key–value pairs that contain arbitrary nonsecret data\. When you include an encryption context in a request to encrypt data, AWS KMS cryptographically binds the encryption context to the encrypted data\. To decrypt the data, you must pass in the same encryption context\. 

Elastic Transcoder uses the same encryption context in all AWS KMS API requests to generate data keys, encrypt, and decrypt\.

```
"service" : "elastictranscoder.amazonaws.com"
```

The encryption context is written to CloudTrail logs to help you understand how a given AWS KMS KMS key was used\. In the `requestParameters` field of a CloudTrail log file, the encryption context looks similar to the following:

```
"encryptionContext": {
  "service" : "elastictranscoder.amazonaws.com"
}
```

For more information about how to configure Elastic Transcoder jobs to use one of the supported encryption options, see [Data Encryption Options](https://docs.aws.amazon.com/elastictranscoder/latest/developerguide/encryption.html) in the *Amazon Elastic Transcoder Developer Guide*\.