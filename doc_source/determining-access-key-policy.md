# Examining the key policy<a name="determining-access-key-policy"></a>

[Key policies](key-policies.md) are the primary way to control access to KMS keys\. Every KMS key has exactly one key policy\.

When a key policy consists of or includes the [default key policy](key-policies.md#key-policy-default-allow-root-enable-iam), the key policy allows IAM administrators in the account to use IAM policies to control access to the KMS key\. Also, if the key policy gives [another AWS account](key-policy-modifying-external-accounts.md) permission to use the KMS key, the IAM administrators in the external account can use IAM policies to delegate those permissions\. To determine the complete list of principals that can access the KMS key, [examine the IAM policies](determining-access-iam-policies.md)\. 

To view the key policy of an AWS KMS [ customer managed key](concepts.md#customer-cmk) or [AWS managed key](concepts.md#aws-managed-cmk) in your account, use the AWS Management Console or the [GetKeyPolicy](https://docs.aws.amazon.com/kms/latest/APIReference/API_GetKeyPolicy.html) operation in the AWS KMS API\. To view the key policy, you must have `kms:GetKeyPolicy` permissions for the KMS key\. For instructions for viewing the key policy for a KMS key, see [Viewing a key policy](key-policy-viewing.md)\.

Examine the key policy document and take note of all principals specified in each policy statement's `Principal` element\. The IAM users, IAM roles, and AWS accounts in the `Principal` elements are those that have access to this KMS key\.

**Note**  
Do not set the Principal to an asterisk \(\*\) in any key policy statement that allows permissions unless you use conditions to limit the key policy\. An asterisk gives every identity in every AWS account permission to use the KMS key, unless another policy statement explicitly denies it\. Users in other AWS accounts just need corresponding IAM permissions in their own accounts to use the KMS key\.

The following examples use the policy statements found in the [default key policy](key-policies.md#key-policy-default) to demonstrate how to do this\.

**Example Policy statement 1**  

```
{
  "Sid": "Enable IAM policies",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:root"},
  "Action": "kms:*",
  "Resource": "*"
}
```
In policy statement 1, `arn:aws:iam::111122223333:root` refers to the AWS account 111122223333\. By default, a policy statement like this one is present in the key policy document when you create a new KMS key with the AWS Management Console\. It is also present when you create a new KMS key programmatically but do not provide a key policy\.  
A key policy document with a statement that allows access to the AWS account \(root user\) enables [IAM policies in the account to allow access to the KMS key](key-policies.md#key-policy-default-allow-root-enable-iam)\. This means that IAM users and roles in the account might have access to the KMS key even if they are not explicitly listed as principals in the key policy document\. Take care to [examine all IAM policies](determining-access-iam-policies.md) in all AWS accounts listed as principals to determine whether they allow access to this KMS key\.

**Example Policy statement 2**  

```
{
  "Sid": "Allow access for Key Administrators",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:user/KMSKeyAdmin"},
  "Action": [
    "kms:Describe*",
    "kms:Put*",
    "kms:Create*",
    "kms:Update*",
    "kms:Enable*",
    "kms:Revoke*",
    "kms:List*",
    "kms:Disable*",
    "kms:Get*",
    "kms:Delete*",
    "kms:ScheduleKeyDeletion",
    "kms:CancelKeyDeletion"
  ],
  "Resource": "*"
}
```
In policy statement 2, `arn:aws:iam::111122223333:user/KMSKeyAdmin` refers to the IAM user named KMSKeyAdmin in AWS account 111122223333\. This user is allowed to perform the actions listed in the policy statement, which are the administrative actions for managing a KMS key\.

**Example Policy statement 3**  

```
{
  "Sid": "Allow use of the key",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:role/EncryptionApp"},
  "Action": [
    "kms:DescribeKey",
    "kms:GenerateDataKey*",
    "kms:Encrypt",
    "kms:ReEncrypt*",
    "kms:Decrypt"
  ],
  "Resource": "*"
}
```
In policy statement 3, `arn:aws:iam::111122223333:role/EncryptionApp` refers to the IAM role named EncryptionApp in AWS account 111122223333\. Principals that can assume this role are allowed to perform the actions listed in the policy statement, which are the cryptographic actions for encrypting and decrypting data with a KMS key\.

**Example Policy statement 4**  

```
{
  "Sid": "Allow attachment of persistent resources",
  "Effect": "Allow",
  "Principal": {"AWS": "arn:aws:iam::111122223333:role/EncryptionApp"},
  "Action": [
    "kms:ListGrants",
    "kms:CreateGrant",
    "kms:RevokeGrant"
  ],
  "Resource": "*",
  "Condition": {"Bool": {"kms:GrantIsForAWSResource": true}}
}
```
In policy statement 4, `arn:aws:iam::111122223333:role/EncryptionApp` refers to the IAM role named EncryptionApp in AWS account 111122223333\. Principals that can assume this role are allowed to perform the actions listed in the policy statement\. These actions, when combined with the actions allowed in **Example policy statement 3**, are those necessary to delegate use of the KMS key to most [AWS services that integrate with AWS KMS](service-integration.md), specifically the services that use [grants](grants.md)\. The `Condition` element ensures that the delegation is allowed only when the delegate is an AWS service that integrates with AWS KMS and uses grants for authorization\.

To learn all the different ways you can specify a principal in a key policy document, see [Specifying a Principal](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html#Principal_specifying) in the *IAM User Guide*\.

To learn more about AWS KMS key policies, see [Key policies in AWS KMS](key-policies.md)\.