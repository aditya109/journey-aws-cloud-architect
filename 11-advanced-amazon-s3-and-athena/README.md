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
- Helpful when you want to share large datasets with other accounts.
- The requester must be authenticated in AWS (cannot be anonymous).
-  

## S3 Event Notification

## S3 Performance

## S3 Select & Glacier Select

## S3 Batch Operations

## Quiz

