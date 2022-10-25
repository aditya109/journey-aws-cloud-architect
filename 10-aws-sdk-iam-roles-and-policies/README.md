# AWS SDK, IAM Roles & Policies

IAM Roles and Policies

AWS Policy Generator 

AWS Policy Simulator

AWS SDK Overview

## AWS EC2 Instance Metadata

- It allows AWS EC2 instances to *learn about themselves*, without using an IAM role for that purpose.

- The URL is [http://169.254.169.254/latest/meta-data]()

  ```bash
  ~?>curl http://169.254.169.254/latest/
  dynamic
  metadata
  userdata
  ```

- You can retrieve the IAM Role name from the metadata, but you can CANNOT retrieve the IAM Policy.

- Metadata = info about the EC2 instance.

## AWS CLI Dry Run

Use `--dry-run` suffixed to the CLI command you are running.

## AWS CLI STS Decode

When we run API calls and they fail, we can get a long error message, which could be decoded using the **STS** command line.

`````` bash
aws sts decode-authorization-message --encoded-message <value>
``````

## AWS CLI Profiles

Use CLI extensions `--profile` when using multiple profiles.

```bash
aws configure --profile <profile-name>
```

## AWS CLI with MFA

## Exponential Back-off and service limit increase 

- API Rate Limits
  - **DescribeInstances** API for EC2 has a limit of 100 calls per seconds.
  - **GetObject** on S3 has a limit of 5500 GET per second per prefix
  - For intermittent errors: implement Exponential back-off
  - For consistent errors: request an API throttling limit increase
- Service Quotas (Service limits)
  - Running on-demand standard instances: 1152 vCPU
  - You can request a service limit increase by opening a ticket
  - You can request a service increase by using the service Quotas API.
- Exponential back-offs
  - If you get `ThrottlingException` intermittently, use exponential back-off.
  - Retry mechanism already included in AWS SDK API calls.
  - Must implement yourself if using the AWS API as-is or in specific cases.
    - Must only implement the retries on <u>5xx server errors</u> and throttling.
    - Do not implement on the 4xx client errors.

## AWS Credentials Provider & Chain

## AWS Signature v4 Signing 

When you call the AWS HTTP API, you sign the request so that AWS can identify you, using your AWS credentials (access key and secret key).

Note: some requests to Amazon S3 don't need to be signed.

If you use the SDK or CLI, the HTTP requests are signed for you.

You should sign an AWS HTTP request using Signature v4 (SigV4).

1. Create canonical request.
2. Create string to sign.
3. Calculate signature.
4. Add signature to request.

SigV4 request examples:

- HTTP Header option
- Query string option

## Quiz

