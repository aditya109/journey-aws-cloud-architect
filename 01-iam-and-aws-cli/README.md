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







