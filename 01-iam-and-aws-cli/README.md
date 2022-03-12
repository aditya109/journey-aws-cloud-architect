# IAM: Users & Groups

IAM = Identify and Access Management, Global service

- **Root account** created by default, shouldn't be used or shared.
- *Users* are people within our organization, and can be grouped.
- *Groups* only contain users, not other groups.
- Users *don't have to belong to a group*, and user *can belong to multiple groups*.

## IAM: Permissions

- **Users or Groups** can be assigned JSON documents called policies.

  ```json
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": "ec2:Describe*",
              "Resource": "*"
          },
          {
              "Effect": "Allow",
              "Action": "elasticloadbalancing:Describe*",
              "Resource": "*"
          },
          {
              "Effect": "Allow",
              "Action": [
                  "cloudwatch:ListMetrics",
                  "cloudwatch:GetMetricsStatistics",
                  "cloudwatch:Describe"
              ],
              "Resource": "*"
          }
      ]
  }
  ```

- These policies define the **permissions** of the users.

- In AWS, you apply the **least privilege principle**: don't give more permissions than a user needs.

- For a user, we can add `Tags`.

For an account, we have an Account ID and also we can provide alias for faster access.

## IAM Policies Inheritance

For a user, the policies availed by him/her is an intersection of all the policies from all the groups of which he is a part of.

### IAM Policies Structure

```json
{
    "Version": "2012-10-17",
    "Id": "S3-Account-Permissions",
    "Statement": [
        {
            "Sid": "1",
            "Effect": "Allow",
            "Principal": {
                "AWS": [
                    "arn:aws:iam:123456789012:root"
                ]
            },
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::mybucket/*"
            ]
        }
    ]
}
```

An IAM Policies Structure

- Consists of 
  - **Version**: policy language version, always include `"2012-10-17"`
  - **Id**: an identifier for the policy (optional)
  - **Statement**: one or more individual statements (required)
- Statements consists of 
  - **Sid**: an identifier for the statement (optional)
  - **Effect**: whether the statement allows or denies access (Allow, Deny)
  - **Principal**: account/user/role to which this policy applied to
  - **Action**: list of actions this policy allow or denies
  - **Resource**: list of resources to which the actions applied to
  - **Condition**: conditions for when this policy is in effect (optional)

There are two types of permission policies which can be attached to a particular user:

1. *Add permissions*: doing it with pre-defined set of permission policies.
2. *Add inline policy*: created our own custom permission policy and add it directly to the user.
3. *Inherited from group*: policies from all the groups the user is part of.

## IAM - User Protection

There are two mechanisms present in AWS for user protection.

1. **IAM - Password Policy** 
   - Strong passwords = higher security for your account.
   - In AWS, a password policy can be set.
     For example,
     - include uppercase letters, lowercase letters, numbers, non-alphanumberic characters.
     - allow all IAM users to change their own passwords.
     - require users to change thier passwords after some time (password expiration).
     - prevent password re-use.
2. MFA Multi Factor Authentication
   - Users have access to your account and can possibly change configurations or delete resources in your AWS account.
   - *Why ?* **You'd want to protect your root accounts and IAM users.**
   - Main benefit of MFA: <u>if a password is stolen or hacked, the account is not compromised.</u>
   - MFA devices options in AWS
     - Virtual MFA devices
       - Google Authenticator
       - Authy

Using `oathtool` for MFA.

1. Goto account > `Security Credentials` > Add a virtual device for MFA.

2. Get the secret and place it in `~/.aws/secrets/my-secret-server` file.

3. Use the following command to generate TOTPs.

   ```
   oathtool --totp --base32 $(cat ~/.aws/secrets/my-secret-server)
   ```

## AWS CLI

How can users access AWS ?

- To access AWS, you have 3 options:
  - AWS Management Console (*protected by password + MFA*)
  - AWS CLI (*protected by access keys*)
  - AWS SDK (*protected by access keys*)
- Users manage their own access keys. *Kindly do not share.*
  Access key ID ~= username
  Secret access key ~= password

## IAM Roles for AWS Services

Common roles:

- EC2 instance roles
- Lambda function roles

# IAM Security Tools

- IAM Credentials Report (account-level)
  - A report that lists all your account's users and the status of their various credentials.
    - To create this report, go to `Security Credentials`  > `Credential report` > `Download credential report`.
- IAM Access Advisor (user-level)
  - Access advisor shows the service permissions granted to a user and when those services were last accessed.
  - You can use this information to revise your policies.
    - IAM > Access Management > Users > your_user > Access Advisor 





