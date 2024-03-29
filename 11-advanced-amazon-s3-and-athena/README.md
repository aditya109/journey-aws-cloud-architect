# Advanced S3

Table of Contents

## S3 Lifecycle rules (With S3 Analytics)

Amazon S3 - Moving between Storage classes

- You can transition objects between storage classes, automatic movement is defined by **Lifecycle Rule**.
- **Transition Actions** - configure objects to transition to another storage class.
  - Move objects to Standard IA class 60 days after creation
  - Move to Glacier for archiving after 6 months.
- **Expiration Actions** - configure objects to expire (delete) after some time.
  - Access log files can be set to delete after a 365 days.
  - Can be used to delete old versions of file (if versioning is enabled)
  - Can be used to delete incomplete multi-part uploads.
- Rules can be created for a certain prefix (example: [s3://mybucket/mp3/*]())
- Rules can be created for a certain objects tags (example: Department: Finance)****

### Examples

| 1.     | Your application on EC2 creates images thumbnails after profile phots are uploaded to Amazon S3. These thumbnails can be easily recreated, and only need to be kept for 60 days. The source images should be able to be immediately retrieved for these 60 days, and afterwards, the user can wait up to 60 days. How would you design this ? |
| ------ | ------------------------------------------------------------ |
| Ans.   | S3 source images be on **Standard**, with a lifecycle configuration to transition them to **Glacier** after 60 days.<br />S3 thumbnails can be on **One-Zone IA**, with a lifecycle configuration to expire them (delete them) after 60 days. |
| **2.** | A rule in your company state that you should be able to recover your deleted S3 objects immediately for 30 days, although this may happen rarely. After this time, and for up to 365 days, deleted objects should be recoverable within 48 days. |
| Ans.   | Enable S3 Versioning in order to have object versions, so that *delted objects* are in fact hidden by a *delete marker* and can be recovered.<br />Transition the *nonconcurrent versions* of the object to **Standard IA**.<br />Transition  afterwards the *nonconcurrent versions* to **Glacier Deep Archive**. |
|        |                                                              |

### S3 Analytics

This helps us decide when to transition objects to the right storage class.

- Recommendations for **Standard** and **Standard IA**.
  - Does not work for One-zone IA or Glacier

- Rules can be created for a certain prefix (example: [s3://mybucket/mp3/*]()) 
- Rules can be created for a certain objects tags (example: Department: Finance)

### Examples

| 1.     | Your application on EC2 creates images thumbnails after profile photos are uploaded to Amazon S3. These thumbnails can be easily recreated, and only need to be kept for 60 days. The source images should be able to be immediately retrieved for these 60 days, and afterwards, the user can wait up to 60 days. How would you design this ? |
| ------ | ------------------------------------------------------------ |
| Ans.   | S3 source images be on **Standard**, with a lifecycle configuration to transition them to **Glacier** after 60 days.<br />S3 thumbnails can be on **One-Zone IA**, with a lifecycle configuration to expire them (delete them) after 60 days. |
| **2.** | A rule in your company states that you should be able to recover your deleted S3 objects immediately for 30 days, although this may happen rarely. After this time, and for upto 365 days, deleted objects should be recoverable within 48 hours. |
| Ans.   | Enable S3 Versioning in order to have object versions, so that *deleted objects* are in fact hidden by a *delete marker* and can be recovered.<br />Transition the *non-current versions* of the objects to **Standard IA**.<br />Transition afterwards the *non-current versions* to **Glacier Deep Archive**. |

### Amazon S3 Analytics - Storage Class Analysis

- Help you decide when to transition objects to the right storage class.
- Recommended for **Standard** and **Standard IA**.
  - Does NOT work for One-Zone IA or Glacier
- Report is updated daily.
- 24 to 48 hours to start seeing data analysis.

## S3 Requester Pays

- In general, bucket owners pay for all Amazon S3 storage and data transfer costs associated with their bucket.
- **With Requester Pays buckets**, the requester instead of the bucket owner pays the cost of the request and the data download from the bucket.
- Helpful when you want to share large data-sets with other accounts.
- The requester must be authenticated in AWS (cannot be anonymous).

## S3 Event Notification

- The events like `S3:ObjectCreated`, `S3:ObjectRemoved`, `S3:ObjectRestore`, `S3:Replication`, ... can be filtered based on object name.

  > *Use-case*: generate thumbnails of images uploaded to S3.

- **Can create as many *S3 events* as desired**.

- S3 event notifications typically deliver events in seconds but can sometimes take a minute or longer.

### Event notifications with Amazon EventBridge

- Advanced filtering options with JSON rules (metadata, object size, name, ....)
- Multiple destinations - ex-step functions, Kinesis Streams/Firehose,...etc
- EventBridge Capabilities - archive, replay events, reliable delivery

## S3 Performance

### Baseline Performance

Amazon S3 automatically scales to high request rates, latency 100-200 ms.

> Any application can achieve at least 3.5k PUT/COPY/POST/DELETE and 5.5k GET/HEAD requests per second per prefix in a bucket.

There are no limits to the number of prefixes in a bucket.
Example (object path => prefix):

- `bucket/folder1/sub1/file` => `/folder1/sub1`
- `bucket/folder1/sub2/file` => `/folder1/sub2`
- `bucket/1/file` => `/1/`
- `bucket/2/file` => `/2/`

If you spread reads across all four prefixes evenly, you can achieve 22k requests per second for GET and HEAD.

#### Upload types

##### Multi-part upload

- Recommended for files > 100 MB
- Can help parallelize uploads (speeds up transfers)

##### S3 Transfer Acceleration

- Increase transfer speed by transferring file to an AWS edge location which will forward the data to the S3 bucket in the target region.
- Compatible with multi-part upload.

### S3 Byte-Range Fetches

- Parallelize GETs by requesting specific byte ranges.
- Better resilience in case of failures.
- Can be used:
  - To speed up downloads.
  - To retrieve only partial data.

## S3 Select & Glacier Select

- Retrieve less data using SQL by performing **server-side filtering**.
- Can filter by rows and columns (simple SQL statements).
- Less network transfer, less CPU cost client-side.

## S3 Batch Operations

- Perform bulk operations on existing S3 objects with a single request, example:
  - modify object metadata & properties
  - copy objects between S3 buckets
  - encrypt unencrypted objects
  - modify ACLs, tags
  - restore objects from S3 Glacier
  - invoke Lambda function to perform custom action on each object
- A job consists of a list of objects, the action to perform and optional parameters
- S3 Batch Operations manages retries, tracks progress, sends completion notifications, generate reports, etc.
- **You can use S3 Inventory to get object list and use S3 Select to filter your objects.**

## Quiz

| Q.     | While you're uploading large files to an S3 bucket using Multi-part Upload, there are a lot of unfinished parts stored in the S3 bucket due to network issues. You are not using these unfinished parts and they cost you money. What is the best approach to remove these unfinished parts? |
| ------ | ------------------------------------------------------------ |
| Ans.   | Use an S3 Lifecycle Policy to automate old/unfinished parts deletion. |
| **Q.** | **You are looking to build an index of your files in S3, using Amazon RDS PostgreSQL. To build this index, it is necessary to read the first 250 bytes of each object in S3, which contains some metadata about the content of the file itself. There are over 100,000 files in your S3 bucket, amounting to 50 TB of data. How can you build this index efficiently?** |
| Ans.   | Create an application that will traverse the S3 buckets, issue a Byte Range Fetch for the first 250 bytes, and store that information in RDS. |
| **Q.** | **While you're uploading large files to an S3 bucket using Multi-part Upload, there are a lot of unfinished parts stored in the S3 bucket due to network issues. You are not using these unfinished parts and they cost you money. What is the best approach to remove these unfinished parts?** |
| Ans.   | Use an S3 Lifecycle Policy to automate old/unfinished parts deletion. |
| **Q.** | **You are looking to build an index of your files in S3, using Amazon RDS PostgreSQL. To build this index, it is necessary to read the first 250 bytes of each object in S3, which contains some metadata about the content of the file itself. There are over 100,000 files in your S3 bucket, amounting to 50 TB of data. How can you build this index efficiently?** |
| Ans.   | Create an application that will traverse the S3 buckets, issue a Byte Range Fetch for the first 250 bytes, and store that information in RDS. |
|        |                                                              |
|        |                                                              |

## S3 Encryption

- You can encrypt objects in S3 buckets using one of 4 methods.

  - Server-side encryption (SSE)

    - SSE with Amazon S3-Managed Keys (SSE-S3) 

      - Encrypts S3 objects using keys handled, managed and owned by AWS
      - Encryption type is AES-256
      - Must set header `"x-amz-server-side-encryption":"AES256"`.
      - **Enabled by default for new buckets & new objects.**

    - SSE with KMS keys stored in AWS KMS (SSE-KMS) 

      - Encryption using keys handled and managed by AWS KMS (Key Management Service)
      - KMS advantages: user control + audit key usage using Cloud Trail.
      - Must set header `"x-amz-server-side-encryption":"aws:kms"`.

      Limitation:

      - If you use SSE-KMS, you may be impacted by KMS limits.
      - When you upload, it calls the **GenerateDataKey** KMS API.
      - When you download, it calls the **Decrypt** KMS API.
      - Count toward the KMS quota per second (5.5k/10k/30k req/s based on region).
      - You may request a quota increase using the Service Quota Console.

    - SSE with Customer-Provided Keys (SSE-C) (

      - SSE using keys fully managed by the customer outside of AWS.
      - AWS S3 does **NOT** store the encryption key you provide.
      - **HTTPS must be used.**
      - Encryption key must provided in HTTP headers, for every HTTP request made.

  - Client-side encryption

    - Use client libraries such as **Amazon S3 Client-side Encryption Library**
    - Clients must encrypt data themselves before sending to Amazon S3.
    - Clients must decrypt data themselves when retrieving from Amazon S3.
    - Customer fully manages the keys and encryption cycle.

- Encryption in-flight/ in-transit (SSL/TLS)

  - Amazon S3 exposes two endpoints:
    - **HTTP Endpoint** - non-encrypted
    - **HTTPS Endpoint** - in-flight encryption (recommended, required for SSE-C, default for most clients)

## S3 Default Encryption

Default Encryption vs Bucket Policies

- SSE-S3 encryption is automatically applied to new objects stored in S3 bucket.
- Optionally, you can *force encryption* using a bucket policy and refuse any API call to PUT an S3 object without encryption headers (SSE-KMS or SSE-C)

**NOTE:** Bucket Policies are evaluated before Default Encryption.

## S3 CORS

### What is CORS ?

It is Cross-Origin Resource Sharing (CORS), where Origin = scheme (protocol) + host (domain) + port. 
For example, https://www.example.com (implied port is 443 for HTTPS, 80 for HTTP)

It is a web-browser based mechanism to allow requests to other requests while visiting the main origin.
For example,  https://www.example.com/app1 and https://www.example.com/app2 are of same origin, whereas 
https://www.example.com and https://other.example.com are from different origins. The request won't be fulfilled unless the other origin allows for the requests, using CORS Headers (example: **Access-Control-Allow-Origin**), which is known during pre-flight reque



## S3 MFA Delete

## S3 MFA Delete

## S3 Access Logs

## S3 Pre-signed URLs

## Glacier Vault Lock & S3 Object Lock
