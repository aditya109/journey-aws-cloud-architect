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





