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

| 1.     | Your application on EC2 cerates images thumbnails after profile phots are uploaded to Amazon S3. These thumbnails can be easily recreated, and only need to be kept for 60 days. The source images should be able to be immediately retrieved for these 60 days, and afterwards, the user can wait up to 60 days. How would you design this ? |
| ------ | ------------------------------------------------------------ |
| Ans.   | S3 source images be on **Standard**, with a lifecycle configuration to transition them to **Glacier** after 60 days.<br />S3 thumbnails can be on **One-Zone IA**, with a lifecycle configuration to expire them (delete them) after 60 days. |
| **2.** | A rule in your company state                                 |
|        |                                                              |
|        |                                                              |



## S3 Requester Pays

## S3 Event Notification

## S3 Performance

## S3 Select & Glacier Select

## S3 Batch Operations

## Quiz

