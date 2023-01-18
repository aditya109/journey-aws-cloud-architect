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

