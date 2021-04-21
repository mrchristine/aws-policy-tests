# AWS IAM Policy Tests

Document how to test EC2 policies with conditions.  

Steps to reproduce:
First create a cross account IAM role defined by Databricks documentation, e.g. `role/e2-testsuite-role`  

In `~/.aws/config`, add a profile that will assume the Databricks cross account role. 
```
[profile iam-dbx-tests]
region = us-east-2
source_profile = default
role_arn = arn:aws:iam::123456789:role/e2-testsuite-role
```
The `source_profile` here defines which AWS IAM role / profile in the same config file that has assume role access to the `role_arn`. Here it's set to the default profile, but could be any defined profile in this file.  
To allow assume role permissions, the cross account IAM role should define a trust relationship to allow this to assume it's role. 

Define the permission on the `default` role to allow assume role: 
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": [
                "arn:aws:iam::123456789:role/e2-testsuite-role"
            ]
        }
    ]
}
```

In the cross account role, edit the trust relationship to allow the default role to use this profile:
```
	{
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789:user/db-test-user"
      },
      "Action": "sts:AssumeRole"
    }
```
This establishes that the user can assume the role, and the role agrees that this user can assume this role. 

Common errors:
```ClientError: An error occurred (AccessDenied) when calling the AssumeRole operation: User: arn:aws:iam::123456789:user/db-test-user is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::123456789:role/e2-testsuite-role```
The reason for this error is because the trust relationship is likely missing from the cross account IAM role. 

```
ClientError: An error occurred (UnauthorizedOperation) when calling the TerminateInstances operation: You are not authorized to perform this operation. Encoded authorization failure message: R3JlYXQgam9iIGRlY29kaW5nIHRoaXMgZXJyb3IgbWVzc2FnZSE=
```
The reasons for this is the IAM role restrictions / conditionals are not allowing the API call defined in this error, e.g. TerminateInstances in this example. 



**Resources:**
* [AWS Docs for Supported Conditions](https://docs.aws.amazon.com/service-authorization/latest/reference/list_amazonec2.html)
* [Databricks E2 Deployment IAM Roles](https://docs.databricks.com/administration-guide/account-api/iam-role.html#language-Your%C2%A0VPC,%C2%A0custom)
* [AWS Trust Policies](https://aws.amazon.com/blogs/security/how-to-use-trust-policies-with-iam-roles/)
