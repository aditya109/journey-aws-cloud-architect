# Amazon S3

Table of Contents



## S3 Overview

Standard usecases:

- Backup and storage
- Disaster recovery
- Archive
- Hybrid cloud storage
- Application hosting
- Media hosting
- Datalakes & big data analytics
- Software delivery 
- Static website

Example, **NASDAQ** stores 7 years of data into S3 Glacier. **Sysco** runs analytics on its data and gain business insights.

### S3 Buckets

- It allows people to store objects (files) in *buckets* (directories).
- Buckets must have a *globally unique name (across all regions and on all accounts)*.
- Buckets are defined at the region level.
- S3 looks like a global service but buckets are created in a region.

> Naming convention:
>
> 1. No uppercase, no underscore.
> 2. 3-63 characters long.
> 3. Not an IP.
> 4. Must start with lowercase letter or number.
> 5. Must **NOT** start with the prefix `xn--`.
> 6. Must **NOT** end with the suffix `-s3alias`.

### S3 Objects

- Objects (files) have a key.

- The key is the FULL path:

  - `s3://my-bucket/my_file.txt`
  - `s3://my-bucket/my_folder_1/another_folder/my_file.txt`

- The key is composed of <span style="color:darkorange">prefix</span> + <span style="color:limegreen">object_name</span>. Example, 
  s3://my-bucket/<span style="color:darkorange">my_folder_1</span>/<span style="color:darkorange">another_folder</span>/<span style="color:limegreen">my_file.txt</span>

  > <span style="color:maroon">**There is no concept of *directories* within buckets; just keys with very long names.**</span>

- Object values are the content of the body:

  - Maximum object size is 5 GB.
  - If uploading > 5 GB, must use *multi-part upload*.

- Metadata (list of text key/value pairs - system or user metadata).

- Tags (Unicode key/value pair <= 10) - useful for security/life-cycle.

- Version ID (if versioning is enabled)

> **Note:** 
>
> - By default, S3 objects cannot be accessed publicly, if not enabled in settings. Although you get a pre-signed URL, which contains a authorization signature in the query parameter in the URL which can be accessed publicly. However, you can not obtained it without having the access to the account (S3 bucket).

## S3 Security: Bucket Policy

- User-based - **IAM Policies** - which API calls should be allowed for a specific user from IAM.
- Resource-based:
  - **Bucket Policies** - bucket wide rules from the S3 console - allows cross account.
  - **Object Access Control List (ACL)** - finer grain (can be disabled).
  - **Bucket Access Control List (ACL)** - less common (can be disabled).

> Note:
>
> An IAM Principal can access an S3 object if:
>
> - The user IAM permissions ALLOW it, or,
> - The resource policy ALLOWS it, and, there's no explicit DENY.

- Encryption: encrypt objects in Amazon S3 using encryption keys.

### S3 Bucket Policies

- These are JSON-based policies with
  - *Resources*: buckets and object
  - *Effect*: Allow/Deny
  - *Actions*: Set of API to Allow or Deny
  - *Principal*: The account or user to apply the policy to 
- We use S3 bucket policy to:
  - Grant public access to the bucket.
  - Force objects to be encrypted at upload.
  - Grant access to another account.

### Which policy to use when ?

1. Public access - use bucket policy
2. User access to S3 - IAM permissions
3. EC2 instance access - use IAM roles
4. Cross account access - use bucket policy

### Other settings

1. **Bucket settings for Block Public Access** - These settings were created to prevent company data leaks.
2. If you know you bucket should never be public, leave these on.
3. Can be set at account level.

**Hands On**

Use policy generator to generate policies.

To specify a buckets, ARN should be bucket_name followed by *, not just bucket name. For example, 
`arm:aws:s3:::my-s3-bucket/*`.

## S3 Website Overview

- S3 can host static websites and have them accessible on the Internet.

- The website URL will be (depending on the region)

  - `http://bucket-name.s3-website-aws-region.amazonaws.com`
  - `http://bucket-name.s3-website.aws-region.amazonaws.com`

  > <span style="color:darkorange">If you get a **403 Forbidden** error, make sure the bucket policy allows public reads.</span>

## S3 Versioning

- You can version your files in Amazon S3.
- It is enabled at the bucket level.
- Same key overwrite will change the `version`: 1,2,3,. ....
- It is best practice to version your buckets:
  - Protect against unintended deletes (ability to restore a version)
  - Easy to roll back to previous version

> Note:
>
> 1. Any file that is not versioned prior to enabling versioning will have version `null`.
> 2. Suspending versioning does not delete the previous versions.

**Hands On**

You can enable Delete Marker while deleting an S3 bucket to restore it later.

## S3 Replication

- Must enable versioning in source and destination buckets.
- Buckets can be in different AWS accounts.
- Copying is asynchronous.
- Must give proper IAM permission to S3.
- After you enable Replication, only new objects are replicated.
- Optionally, you can replicate existing objects using **S3 Batch Replication**
  - Replicates existing objects and objects that failed replication.
- For DELETE operations:
  - **Can replicate delete markers** from source to target (optional settings)
  - Deletions with a version ID are not replicated (to avoid malicious deletes)
- There is no *chaining* of replication, meaning if bucket 1 has replication into bucket 2, which has replication into bucket 3, then objects created in bucket 1 are not replicated to bucket 3.
- Types of replication:
  - **Cross region replication (CRR)**
    - Use-case: compliance, lower latency access, replication across accounts.
  - **Same region replication (SRR)**
    - Use-case: log aggregation, live replication between production and test accounts.

### Additional Replication options

1. **Replication Time Control (RTC)** - It replicates 99.99% of new objects within 15 minutes and provides replication metrics and notifications. (Chargeable)
2. **Replication metrics and notifications** - Monitor the progress of your replication rule through Cloudwatch Metrics. (Chargeable)
3. **Delete marker replication** - Delete markers created by S3 delete operations will be replicated. Delete markers created by lifecycle rules are not replicated.
4. **Replica modification sync** - Replicate metadata changes made to replicas in this bucket to the destination bucket.

## S3 Storage Classes Overview

- Types of S3 classes:

  - General Purpose

    - ***Amazon S3 Standard - General Purpose***

      - 99.99 % availability

      - used for frequently accessed data

      - low latency and high throughput

      - sustain 2 concurrent facility failures.

        > Use-cases: big-data analytics, mobile & gaming applications, content distribution, etc.

  - Infrequent Access

    - for data that is less frequently accessed, but requires rapid access when needed

    - lower cost than S3 standard.

    - types of IA classes:

      - ***Amazon S3 Standard - Infrequent Access (IA)***

      - 99.9 % availability

        > Use-cases: disaster recovery, backups, etc.

      - ***Amazon S3 One Zone - Infrequent Access (IA)*** 

        - High durability 99.999999999 % in a single AZ; data lost when AZ is destroyed.

        - 99.5% availability

          > Use-cases: storing secondary backup copies of on-premises data, or data you can recreate etc.

    - Glacier

      - low-cost object storage meant for archiving/backup

      - pricing: price for storage + object retrieval cost.

      - types:

        - ***Amazon S3 Glacier - Instant Retrieval*** 
          - retrieval in ms, great for data accessed once a quarter
          - minimum storage duration of 90 days.

        - ***Amazon S3 Glacier - Flexible Retrieval***
          - expedited (1 to 5 mins), standard (3-5 hours), bulk (5-12 hours) - free
          - minimum storage duration of 90 days

        - ***Amazon S3 Glacier - Deep Archive***
          - standard (12 hours), bulk (48 hours)
          - minimum storage duration of 180 days

  - ***Amazon S3 Intelligent Tiering***

    - Small monthly monitoring and auto-tiering fee.
    - Movies objects automatically between access tiers based on usage.
    - There are no retrieval charges in S3 intelligent tiering.
      - *Frequent access tier (automatic)*: default tier
      - *Infrequent access tier (automatic)*: objects not accessed for 30 days
      - *Archive instant access tier (automatic)*: objects not accessed for 90 days
      - *Archive access tier (optional)*: configurable from 90 days to 700+ days
      - *Deep archive access tier (optional)*: configurable from 180 days to 700+ daysS

  - They can move between classes manually or using S3 lifecycle configurations.

### S3 Durability and Availability

- Durability:

  - High (11 9's or 99.999999999 %) of objects across multiple AZ.

    > If you store 10,000,000 objects with Amazon S3, you can on average expect to incur a loss of a single object once every 10000 years.

  - Same for all storage classes.

- Availability:

  - Measures how readily available a service is.
  - Varies depending on storage class.
  - Example: S3 standard has 99.99 % availability = not available 53 minutes a year.

## Quiz