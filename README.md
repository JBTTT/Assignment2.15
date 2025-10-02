Assignment 2.15

Answer the following:

1. What is needed to authorize your EC2 to retrieve secrets from the AWS Secret Manager?

  An Amazon EC2 instance does not inherently possess permissions to interact with AWS services. In particular, the retrieval of sensitive information from AWS Secrets Manager requires explicit authorization mechanisms to be established. These mechanisms are based on the principle of least privilege, ensuring that only approved actions are permitted.

  Role of IAM in Permission Delegation
  The mechanism for delegating such permissions is the AWS Identity and Access Management (IAM) service. Within IAM, a role is created to act as a temporary identity that EC2 instances can assume. This role is defined through two critical policy components:
  •	Trust Policy: Specifies which AWS service is authorized to assume the role. In the context of EC2, this policy designates the EC2 service as the trusted principal.
  •	Permissions Policy: Defines the specific actions and resources accessible to the role. For Secrets Manager integration, the policy commonly includes the action GetSecretValue restricted to the particular secret required by the application.

  Binding the Role to the Instance via an Instance Profile
    An EC2 instance does not directly accept an IAM role. Instead, the role is associated with the instance through an instance profile. The instance profile acts as a bridge, ensuring that when the virtual machine is launched, the role’s permissions are automatically available within the runtime environment of the instance.

  Credential Delivery through Instance Metadata Service
  Once the instance is operational, AWS automatically provisions temporary security credentials linked to the IAM role. These credentials are not stored in a persistent manner; rather, they are exposed to the instance through the Instance Metadata Service (IMDS). The credentials are short-lived and rotate automatically, thereby reducing the risk of compromise.

  Programmatic Access to Secrets
  Applications running on the EC2 instance can subsequently request secrets from AWS Secrets Manager. This is achieved through the AWS SDKs or the AWS Command Line Interface (CLI). These tools natively retrieve the temporary credentials from IMDS and use them transparently to authenticate API requests to Secrets Manager. Upon successful verification of permissions, Secrets Manager responds with the secret value, which can then be consumed by the application.

  Security Considerations
  Several best practices accompany this design pattern:
  •	Least Privilege: Policies should restrict access strictly to the specific secrets required.
  •	Use of IMDSv2: Enforcing the most recent version of the Instance Metadata Service mitigates the risk of metadata-related exploits.
  •	Avoidance of Static Credentials: Long-term IAM user access keys must not be embedded in EC2 instances or application code.
  •	Secret Rotation: AWS Secrets Manager supports automated rotation of secrets, which should be enabled to minimize exposure risk.

  The secure retrieval of secrets from AWS Secrets Manager by an EC2 instance is predicated upon the creation of an IAM role with both a trust and permissions policy, the association of this role with the instance through an instance profile, and the automatic provision of temporary credentials via the Instance Metadata Service. Applications operating within the instance utilize these credentials indirectly through the AWS SDKs or CLI to obtain secret values. This approach exemplifies a secure, ephemeral, and policy-driven mechanism for managing sensitive information in cloud environments.
  The step-by-step procedures are as follows:
  Application on EC2 → IAM Role (Trust + Permissions) → Instance Profile → IMDS (Temporary Credentials) → AWS SDK/CLI → Secrets Manager → Secret returned to application

2. Derive the IAM policy (i.e. JSON)?

  To authorize an EC2 instance to retrieve secrets from AWS Secrets Manager, the IAM role attached to that instance needs a policy that grants access to the relevant APIs and secret resources.
  
  Sample IAM Policy for a Single Secret
  Supposing that the secret name is prod/cart-service/credentials, its ARN would typically appear as follow:
    arn:aws:secretsmanager:us-east-1:<account-id>:secret:prod/cart-service/credentials-<random>
    
    Here is the IAM policy:

    {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "secretsmanager:GetSecretValue",
          "secretsmanager:DescribeSecret"
        ],
        "Resource": "arn:aws:secretsmanager:us-east-1:<account-id>:secret:prod/cart-service/credentials-*"
      }
      ]
    }

  Policy for All Secrets (less restrictive)
  If multiple secrets are needed, you can generalize to allow all:

  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "secretsmanager:GetSecretValue",
          "secretsmanager:DescribeSecret"
        ],
        "Resource": "*"
      }
    ]
  }

  If Using a Customer KMS Key
  Add a statement for KMS decryption:

  { 
    "Effect": "Allow",
    "Action": "kms:Decrypt",
    "Resource": "arn:aws:kms:us-east-1:<account-id>:key/<kms-key-id>",
    "Condition": {
      "StringEquals": {
        "kms:ViaService": "secretsmanager.us-east-1.amazonaws.com"
      }
    }
  }


3.  Using the secret name prod/cart-service/credentials, derive a sensible ARN as the specific resource for access

  The ARN format for a Secrets Manager secret is:

    arn:aws:secretsmanager:<region>:<account-id>:secret:<secret-name>-<6_random_chars>

  For the secret name prod/cart-service/credentials, a sensible ARN would be:

    arn:aws:secretsmanager:ap-southeast-1:123456789012:secret:prod/cart-service/credentials-*

  Where:
    ap-southeast-1 = Singapore region (adjust if your secret is elsewhere).
    123456789012 = Your AWS Account ID.
    prod/cart-service/credentials = Your provided secret name.
    -* = required wildcard suffix to match the auto-generated random string AWS appends to the secret name (e.g., ...credentials-a1b2c3).

In summary, here’s the main steps: 
i)	Create an IAM Role for EC2.
ii)	Trust policy allows EC2 to assume it.
iii)	Attach an IAM policy with secretsmanager:GetSecretValue for the ARN 
      arn:aws:secretsmanager:<region>:<account-id>:secret:prod/cart-service/credentials-*.
iv)	Attach role to EC2 instance.


  
