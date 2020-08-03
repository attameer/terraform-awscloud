# HashiCorp - Terraform AWS Cloud Deployment (pre-release)
## HashiCorp Terraform - Infrastructure Management

![GitHub Actions - Terraform](https://github.com/emvaldes/terraform-awscloud/workflows/GitHub%20Actions%20-%20Terraform/badge.svg)

It's imperative that these GitHub Secrets are set:

```bash
AWS_ACCESS_KEYPAIR:    Terraform AWS KeyPair (PEM file).
AWS_ACCESS_KEY_ID:     Terraform AWS Access Key-Id (e.g.: AKIA2...VT7DU).
AWS_DEFAULT_ACCOUNT:   The AWS Account number (e.g.: 123456789012).
AWS_DEFAULT_PROFILE:   The AWS Credentials Default User (e.g.: default).
AWS_DEFAULT_REGION:    The AWS Default Region (e.g.: us-east-1)
AWS_DEPLOY_TERRAFORM:  Enable|Disable (true|false) terraform deployment
AWS_SECRET_ACCESS_KEY: Terraform AWS Secret Access Key (e.g.: zBqDUNyQ0G...IbVyamSCpe).
```

### The following features described here are not really scalable and will need to be reviewed.

The **AWS_ACCESS_KEYPAIR** is a GitHub Secret used to auto-populate the ***~/access-keypair*** file for post-deployment configurations.

In the event of needing to target a different account, change it in the GitHub Secrets **AWS_DEFAULT_ACCOUNT**. Keep in mind that both **AWS_SECRET_ACCESS_KEY** and **AWS_ACCESS_KEY_ID** are account specific.<br>

There is no need to midify the GitHub Secret **AWS_DEFAULT_PROFILE** as there is only one section defined in the ~/.aws/credentials file. If a specific AWS Region is required, then update the **AWS_DEFAULT_REGION** but keep in mind that any concurrent build will be pre-set.

The logical switch **AWS_DEPLOY_TERRAFORM** is set to enable or disable the deployment of the terraform plan is a safety messure to ensure that a control-mechanism is in place.

**Note**: In addition to these basic/core requirements, it's important that a key-name **terraform** be created/active in AWS as it's hardcoded in this prototype. I will find a more efficient solution to this.

Please, make sure your **AWS IAM Policy** allows for something like this and enforce the appropriate ***User Permissions Boundary***:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::terraform-states-123456789012",
                "arn:aws:s3:::terraform-states-123456789012/*"
            ]
        }
    ]
}
```

I would also recommend that you append an ***AWS IAM Inline Policy*** to your **terraform** ***AWS IAM User account***:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "StmtCustomIAmPolicy",
            "Effect": "Deny",
            "Action": [
                "iam:AttachRolePolicy",
                "iam:UpdateAccessKey",
                "iam:DetachUserPolicy",
                "iam:CreateLoginProfile",
                "ec2:RequestSpotInstances",
                "organizations:InviteAccountToOrganization",
                "iam:AttachUserPolicy",
                "lightsail:Update*",
                "iam:ChangePassword",
                "iam:DeleteUserPolicy",
                "iam:PutUserPolicy",
                "lightsail:Create*",
                "lambda:CreateFunction",
                "lightsail:DownloadDefaultKeyPair",
                "iam:UpdateUser",
                "organizations:CreateAccount",
                "iam:UpdateAccountPasswordPolicy",
                "iam:CreateUser",
                "lightsail:Delete*",
                "iam:AttachGroupPolicy",
                "ec2:StartInstances",
                "iam:PutUserPermissionsBoundary",
                "iam:PutGroupPolicy",
                "lightsail:Start*",
                "lightsail:GetInstanceAccessDetails",
                "iam:CreateAccessKey",
                "organizations:CreateOrganization"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

I would also setup the target ***AWS S3 Bucket Policy*** based on these standard configurations:

```json
{
    "Version": "2012-10-17",
    "Id": "PolicyTerraformS3Bucket",
    "Statement": [
        {
            "Sid": "StmtTerraformS3Bucket",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::123456789012:user/terraform"
            },
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::terraform-states-123456789012/*"
        }
    ]
}
```

Block ALL Bucket public access (bucket settings: ON)
<ol>
<li>Block public access to buckets and objects granted through new access control lists (ACLs)</li>
<li>Block public access to buckets and objects granted through any access control lists (ACLs)</li>
<li>Block public access to buckets and objects granted through new public bucket or access point policies</li>
<li>Block public and cross-account access to buckets and objects through any public bucket or access point policies</li>
</ol>

**Reference**: This project is based on the original training materials from [PluralSight](https://www.pluralsight.com).<br />
[Terraform - Getting Started](https://app.pluralsight.com/library/courses/getting-started-terraform) by [Ned Bellavance](https://app.pluralsight.com/profile/author/edward-bellavance)
