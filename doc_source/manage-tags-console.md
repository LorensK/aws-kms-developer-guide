# Managing KMS key tags in the console<a name="manage-tags-console"></a>

You can add tags to a KMS key when you [create the KMS key](create-keys.md) in the AWS KMS console\. You can also use the **Tags** tab in the console to add, edit, and delete tags on customer managed keys To add, edit, view, and delete tags for a KMS key, you must have the required permissions\. For details, see [Controlling access to tags](tag-permissions.md)\.

## Add tags while creating a KMS key<a name="tag-on-create"></a>

To add tags when creating a KMS key in the console, you must have `kms:TagResource` permission in an IAM policy in addition to the permissions required to create KMS keys and view KMS keys in the console\. At a minimum, the permission must cover all KMS keys in the account and Region\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot manage the tags of an AWS managed key\)

1. Choose the key type, then choose **Next**\.

1. Enter an alias and optional description\.

1. Enter a tag key and, optionally, a tag value\. To add additional tags, choose **Add tag**\. To delete a tag, choose **Remove**\. When you're done tagging your new KMS key, choose **Next**\.

1. Finish creating your KMS key\.

## View and manage tags on existing KMS keys<a name="tag-existing"></a>

To add, view, edit, and delete tags in the console, you need tagging permission on the KMS key\. You can get this permission from the key policy for the KMS key or, if the key policy allows it, from an IAM policy that includes the KMS key\. You need these permissions in addition to the permissions to view KMS keys in the console\.

1. Sign in to the AWS Management Console and open the AWS Key Management Service \(AWS KMS\) console at [https://console\.aws\.amazon\.com/kms](https://console.aws.amazon.com/kms)\.

1. To change the AWS Region, use the Region selector in the upper\-right corner of the page\.

1. In the navigation pane, choose **Customer managed keys**\. \(You cannot manage the tags of an AWS managed key\)

1. You can use the table filter to display only KMS keys with particular tags\. For details, see [Sorting and filtering your KMS keys](viewing-keys-console.md#viewing-console-filter)\.

1. Select the check box next to the alias of a KMS key\.

1. Choose **Key actions**, **Add or edit tags**\.

1. On the details page for KMS key, choose the **Tags** tab\.
   + To create your first tag, choose **Create tag**, type a tag key \(required\) and tag value \(optional\), and then choose **Save**\.

     If you leave the tag value blank, the actual tag value is a null or empty string\.
   + To add a tag, choose **Edit**, choose **Add tag**, type a tag key and tag value, and then choose **Save**\.
   + To change the name or value of a tag, choose **Edit**, make your changes, and then choose **Save**\.
   + To delete a tag, choose **Edit**\. On the tag row, choose **Remove**, and then choose **Save**\.

1. To save your changes, choose **Save changes**\.