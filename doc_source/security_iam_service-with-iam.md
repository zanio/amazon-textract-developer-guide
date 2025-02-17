# How Amazon Textract Works with IAM<a name="security_iam_service-with-iam"></a>

Before you use IAM to manage access to Amazon Textract, you should understand what IAM features are available to use with Amazon Textract\. To get a high\-level view of how Amazon Textract and other AWS services work with IAM, see [AWS Services That Work with IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_aws-services-that-work-with-iam.html) in the *IAM User Guide*\.

**Topics**
+ [Amazon Textract Identity\-Based Policies](#security_iam_service-with-iam-id-based-policies)
+ [Amazon Textract Resource\-Based Policies](#security_iam_service-with-iam-resource-based-policies)
+ [Authorization Based on Amazon Textract Tags](#security_iam_service-with-iam-tags)
+ [Amazon Textract IAM Roles](#security_iam_service-with-iam-roles)

## Amazon Textract Identity\-Based Policies<a name="security_iam_service-with-iam-id-based-policies"></a>

With IAM identity\-based policies, you can specify allowed or denied actions and resources and the conditions under which actions are allowed or denied\. Amazon Textract supports specific actions, resources, and condition keys\. To learn about all of the elements that you use in a JSON policy, see [IAM JSON Policy Elements Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html) in the *IAM User Guide*\.

### Actions<a name="security_iam_service-with-iam-id-based-policies-actions"></a>

Administrators can use AWS JSON policies to specify who has access to what\. That is, which **principal** can perform **actions** on what **resources**, and under what **conditions**\.

The `Action` element of a JSON policy describes the actions that you can use to allow or deny access in a policy\. Policy actions usually have the same name as the associated AWS API operation\. There are some exceptions, such as *permission\-only actions* that don't have a matching API operation\. There are also some operations that require multiple actions in a policy\. These additional actions are called *dependent actions*\.

Include actions in a policy to grant permissions to perform the associated operation\.

Asynchronous actions in Amazon Textract require two action permissions to be given, one for Start actions and one for Get actions\. Additionally, if you are using an Amazon S3 bucket to pass documents, you will need to grant your account read access\.

In Amazon Textract, all policy actions start with: `textract:`\. For example, to grant someone permission to run an Amazon Textract operation with the Amazon Textract `AnalyzeDocument` operation, you include the `textract:AnalyzeDocument` action in their policy\. Policy statements must include either an `Action` or `NotAction` element\. Amazon Textract defines its own set of actions that describe tasks that you can perform with this service\.

To specify multiple actions in a single statement, separate them with commas as follows\.

```
"Action": [
      "textract:action1",
      "textract:action2"
```

You can specify multiple actions using wildcards \(\*\)\. For example, to specify all actions that begin with the word `Describe`, include the following action\.

```
"Action": "textract:Describe*"
```



For a list of Amazon Textract actions, see [Actions Defined by Amazon Textract](https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazontextract.html#amazontextract-actions-as-permissions) in the *IAM User Guide*\.

### Resources<a name="security_iam_service-with-iam-id-based-policies-resources"></a>

Administrators can use AWS JSON policies to specify who has access to what\. That is, which **principal** can perform **actions** on what **resources**, and under what **conditions**\.

The `Resource` JSON policy element specifies the object or objects to which the action applies\. Statements must include either a `Resource` or a `NotResource` element\. As a best practice, specify a resource using its [Amazon Resource Name \(ARN\)](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html)\. You can do this for actions that support a specific resource type, known as *resource\-level permissions*\.

For actions that don't support resource\-level permissions, such as listing operations, use a wildcard \(\*\) to indicate that the statement applies to all resources\.

```
"Resource": "*"
```

Amazon Textract does not support specifying resource ARNs in a policy\.

### Condition Keys<a name="security_iam_service-with-iam-id-based-policies-conditionkeys"></a>

Administrators can use AWS JSON policies to specify who has access to what\. That is, which **principal** can perform **actions** on what **resources**, and under what **conditions**\.

The `Condition` element \(or `Condition` *block*\) lets you specify conditions in which a statement is in effect\. The `Condition` element is optional\. You can create conditional expressions that use [condition operators](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html), such as equals or less than, to match the condition in the policy with values in the request\. 

If you specify multiple `Condition` elements in a statement, or multiple keys in a single `Condition` element, AWS evaluates them using a logical `AND` operation\. If you specify multiple values for a single condition key, AWS evaluates the condition using a logical `OR` operation\. All of the conditions must be met before the statement's permissions are granted\.

 You can also use placeholder variables when you specify conditions\. For example, you can grant an IAM user permission to access a resource only if it is tagged with their IAM user name\. For more information, see [IAM policy elements: variables and tags](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html) in the *IAM User Guide*\. 

AWS supports global condition keys and service\-specific condition keys\. To see all AWS global condition keys, see [AWS global condition context keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\.

Amazon Textract does not provide any service\-specific condition keys, but it does support using some global condition keys\. For a list of all AWS global condition keys, see [AWS Global Condition Context Keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html) in the *IAM User Guide*\.

### Examples<a name="security_iam_service-with-iam-id-based-policies-examples"></a>



To view examples of Amazon Textract identity\-based policies, see [Amazon Textract Identity\-Based Policy Examples](security_iam_id-based-policy-examples.md)\.

## Amazon Textract Resource\-Based Policies<a name="security_iam_service-with-iam-resource-based-policies"></a>

Amazon Textract does not support resource\-based policies\.

## Authorization Based on Amazon Textract Tags<a name="security_iam_service-with-iam-tags"></a>

Amazon Textract does not support tagging resources or controlling access based on tags\.

## Amazon Textract IAM Roles<a name="security_iam_service-with-iam-roles"></a>

An [IAM role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) is an entity within your AWS account that has specific permissions\.

### Using Temporary Credentials with Amazon Textract<a name="security_iam_service-with-iam-roles-tempcreds"></a>

You can use temporary credentials to sign in with federation, assume an IAM role, or to assume a cross\-account role\. You obtain temporary security credentials by calling AWS STS API operations such as [AssumeRole](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html) or [GetFederationToken](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetFederationToken.html)\. 

Amazon Textract supports using temporary credentials\. 

### Service\-Linked Roles<a name="security_iam_service-with-iam-roles-service-linked"></a>

[Service\-linked roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-linked-role) allow AWS services to access resources in other services to complete an action on your behalf\. Service\-linked roles appear in your IAM account and are owned by the service\. An IAM administrator can view but not edit the permissions for service\-linked roles\.

Amazon Textract does not support service\-linked roles\.

**Note**  
As Amazon Textract does not support service\-linked roles, it does not support AWS service principals\. For more information on service principals, see [AWS service principals](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_principal.html#principal-services) in the *IAM User Guide*

### Service Roles<a name="security_iam_service-with-iam-roles-service"></a>

This feature allows a service to assume a [service role](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_terms-and-concepts.html#iam-term-service-role) on your behalf\. This role allows the service to access resources in other services to complete an action on your behalf\. Service roles appear in your IAM account and are owned by the account\. This means that an IAM administrator can change the permissions for this role\. However, doing so might break the functionality of the service\.

Amazon Textract supports service roles\. 

If you are using a service role, you should ensure that your account is secure by limiting the scope of Amazon Textract's access to just the resources you are using\. This can be done by attaching a trust policy to your IAM service role\. For information on how to do this, see [Cross\-service confused deputy prevention](https://docs.aws.amazon.com/textract/latest/dg/cross-service-confused-deputy-prevention.html)\.