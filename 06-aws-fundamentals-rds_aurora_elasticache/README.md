# RDS + Aurora + ElastiCache

## Table of Contents

## RDS Overview

RDS or Relational Database Service, is a managed DB service for DB use SQL as a query language, allowing you to create databases in the cloud that are managed by AWS.

- Postgres
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- Aurora (AWS Proprietary database)

<strong>Advantage over using RDS versus deploying DB on EC2</strong>

1. RDS is a managed service:
   - Automated provisioning, OS patching
   - Continuous backups and restore to specific timestamp (<span style="color:orange">Point in Time Restore</span>).
   - Monitoring dashboards.
   - Read replicas for improved read performance.
   - Multi AZ setup for Disaster Recovery.
   - Maintenance windows for upgrades.
   - Scaling capability (vertical and horizontal)
   - Storage backed by EBS (`gp2` or `io1`)
2. <span style="color:red">But you can't SSH into your instances.</span>

<strong>RDS Backups</strong>

1. Backups are automatically enabled in RDS.
2. Automated backups:
   - Daily full backup of the database (during the maintenance window).
   - Transaction logs are backed-up by RDS every 5 minutes.
     - This gives us the ability to restore to any point in time (from oldest backup to 5 minutes ago).
   - 7 days retentions (can be increased to 35 days).
3. DB Snapshots
   - Manually triggered by the user.
   - Retention of backup for as long as you want.

<strong>RDS - Storage Auto Scaling</strong>

1. Helps you increase storage on your RDS DB instance dynamically.
2. When RDS detects you are running out of free database storage, it scales automatically.
3. Avoid manually scaling your database storage.
4. You have to set **Maximum Storage Threshold** (maximum limit for DB storage).
5. Automatically modify storage if:
   - Free storage is less than 10% of allocated storage.
   - Low-storage lasts at least 5 minutes.
   - 6 hours have passed since last modification.
6. Useful for application with **unpredictable workloads**.
7. Supports all RDS database engines (MariaDB, MySQL, PostgreSQL, SQL Server, Oracle)

### RDS Read Replicas vs Multi AZ

#### RDS Read Replicas for read scalability

- We can create up to 5 read replicas.
- Within AZ, cross AZ or cross-region.
- Replication is **Async,** so reads are eventually consistent.
- Replicas can be promoted to their own DB.
- Applications must update the connection string to leverage read replicas.

##### Use-case

You have prod database that is taking on normal load, and want to run a reporting application to run some analytics.
*You can create a Read replica to run the new workload there, the production application is unaffected.*

<span style="color:red">Read replicas should only be used for SELECT(=read) only kin of statements (not INSERT, UPDATE, DELETE)</span>

#### RDS Read Replicas - Network Cost

- In AWS there's a network cost when data goes from one AZ to another. </br>
  **Cross Region** = <span style="color:red">$</span>
- For RDS Read Replicas within the same region, you don't pay that fee. </br>
  **Same Region** = <span style="color:limegreen">Free</span>

#### RDS Multi AZ (Disaster Recovery)

- SYNC replication
- One DNS name - automatic app failover to standby
- Increase availability 
- Failover in case of loss of AZ, loss of network, instance or storage failure
- No manual intervention in apps
- Not used for scaling.

> Read replicas be setup as Multi AZ for Disaster Recovery (DR)

#### RDS - From Single-AZ to Multi-AZ

- Zero downtime operation (no need to stop the DB)
- Just click on *modify* for the database.
- The following happens internally:
  - A snapshot is taken.
  - A new database is restored from the snapshot in a new AZ.
  - Synchronization is established between the two databases.

**Hands-On**

- Go to RDS > Create database.
- Choose *database creation method* either:
  - *Standard create* - provide detailed changes in configuration options.
  - *Easy create* - use recommended best-practice configurations.
- Next, choose *Configuration*:
  - Choose *Engine options*:
    - Postgres
    - MySQL
    - MariaDB
    - Oracle
    - Microsoft SQL Server
    - Aurora (AWS Proprietary database)
  - Select *MySQL Edition and Version*.
- Next, within *Templates*, choose your use case:
  - Production
  - Dev/Test
  - Free Tier
- Next, within *Settings*:
  - Provide *DB instance identifier*
  - Under *Credential settings*, 
    - Give a master username.
    - Give a master password.
- Next, within *DB instance class*, choose:
  - Standard classes (m classes)
  - Memory optimized classes (r and x classes)
  - Burstable classes (t classes)
- Next, within *Storage*, choose:
  - Select a *Storage type*.
  - Provide *Allocated storage*.
  - Provide *Enable storage autoscaling*.
  - Maximum storage threshold.
- Next, within *Availability & duration*, select for Multi-AZ deployment, it can either be:
  - *Do not create a standby instance*.
  - *Create a standby instance (recommended for production usage, creates a standby in a different AZ to provide data redundancy, eliminate I/O freezes, minimum latency spikes during system backups.)* 
- Next, within *Connectivity*:
  - Provide *VPC*.
  - Provide *Subnet group*.
  - Select whether or not you want to provide *Public access*.
  - Provide *VPC security group*.
  - Provide *AZ* (preference).
- Next, within *Database authentication*, select Database authentication options:
  - Password authentication (*Authenticates using database passwords*)
  - Password and IAM Database authentication (*Authentication using database password and user credentials through AWS IAM roles*)
  - Password and Kerberos authentication (*Choose a directory in which you want to allow authorized users to authenticate with this DB instance using Kerberos Authentication*)
- Next, within *Additional configuration*:
  - We have *Database options*, for providing:
    - *Initial database name*
    - *DB parameter group*
    - *Option group*
  - We have *Backup*:
    - Select *Enable automatic backups*.
    - Provide *Backup retention period.*
    - Select *Backup window.*
    - Enable *Cop tags to snapshots.*
  - We have *Monitoring*:
    - Select *Enable Enhanced monitoring*.
    - Select *Granularity* period.
    - Select *Monitoring Role*.
  - For *Log exports*, select favored log types to publish to Amazon CloudWatch logs.
    - Audit log
    - Error log
    - General log
    - Slow query log
  - For *Maintenance*,
    - Select *enable auto minor version upgrade*.
    - Select Maintenance window.
  - For *Deletion protection*, select *enable deletion protection*.
- Create RDS.

### RDS Encryption + Security

- At rest encryption,

  - Possibility to encrypt the master & read replicas with AWS KMS - AES-256 encryption.
  - <span style="color:orange">Encryption has to be defined at launch time.</span>
  - *If the master is not encrypted, the read replicas <u>cannot</u> be encrypted.*
  - Transparent Data Encryption (TDE) available for Oracle and SQL Server.

- In-flight encryption,

  - SSL encryption to encrypt data to RDS in flight.

  - Provide SSL options with trust certificate when connecting to database.

  - To <u>enforce</u> SSL:

    - PostgreSQL: `rds.force_ssl=1` in the AWS RDS Console (Parameter Groups)

    - MySQL: Within the DB:
      ```sql
      GRANT USAGE ON *.* TO 'mysqluser'@'%' REQUIRE SSL;
      ```

#### RDS Encryption Operations

- **Encrypting RDS backups**
  - Snapshots of un-encrypted RDS databases are un-encrypted, and vice versa.
  - We can copy a snapshot into an encrypted one.
- **To encrypt an un-encrypted RDS database**
  - Create a snapshot of the un-encrypted database.
  - Copy the snaphot and enable encryption for the snapshot.
  - Restore the database from the encrypted snapshot.
  - Migrate applications to the new database, and delete the old database.

#### RDS Security - Network & IAM

Network security

- RDS databases are usually deployed within a private subnet, not in a public one.
- RDS security works by leveraging security groups (same as EC2 instances) - it controls which IP/security group can **communicate** with RDS.

Access Management

- IAM policies help control who can **manage** AWS RDS (through the RDS API)
- Traditional username and password can be used to **login into** the database
- IAM-based authentication can be used to log into RDS MySQL & PostgreSQL

#### RDS - IAM Authentication

- IAM database authentication works with **MySQL** and **PostgreSQL**
- You don't need a password, just an authentication token obtained through IAM & RDS API calls.
- Auth token has a lifetime of 15 minutes.
  ![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/06-aws-fundamentals-rds_aurora_elasticache/assets/rds-iam-authentication.svg)

- Benefits:
  - Network in/out must be encrypted using SSL
  - IAM to centrally manage users instead of DB
  - Can leverage IAM Roles and EC2 instance profiles for easy integration.

## Amazon Aurora

- Aurora is a proprietary technology from AWS (not open sourced).
- Postgres and MySQL are both supported as Aurora DB (that means your drivers will work as if Aurora was a Postgres or MySQL database)
- Aurora is *AWS cloud optimized* and claims 5x performance improvement over MySQL on RDS, over 3x the performance of Postgres on RDS.
- Aurora storage automatically grows in increments of 10GB, up to 128 GB.
- Aurora can have 15 replicas while MySQL has 5, and the replication process is faster (sub 10 ms replica lag).
- Failover in Aurora is instantaneous. It's natively highly available.
- Aurora costs more than RDS (20% more) - but is more efficient.

#### High availability and Read Scaling

- It maintains 6 copies of your data across 3 AZ:
  - 4 copies out of 6 needed for writes
  - 3 copies out of 6 needed for reads
  - Self healing with peer-to-peer replication
  - Storage is striped across 100s of volumes.
- One Aurora instance takes writes (master)
- Automated failover for master in less than 30 seconds.
- Master + up to 15 Aurora Read Replicas serve reads
- Support for Cross Region Replication

#### Aurora DB Cluster

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/06-aws-fundamentals-rds_aurora_elasticache/assets/aurora-db-cluster.svg)

#### Features of Aurora

- Automatic fail-over
- Backup and Recovery
- Isolation and security
- Industry compilance
- Push-button scaling
- Automated patching with zero downtime.
- Advanced monitoring
- Routine Maintenance
- Backtrack: restore data at any point in time without using backups.

#### Aurora Security

- Similar to RDS because it uses the same engine.
- Encryption at rest using KMS.
- Automated backups, snapshots and replicas are also encrypted.
- Encryption in flight using SSL (same process as MySQL or PostgreSQL).
- **Possibility to authenticate using IAM token (same method as RDS)**.
- You are responsible for protecting the instance using security groups.
- <span style="color:red">You can't SSH.</span>

**Hands-On**

- Go to RDS > Create database.

- Choose *database creation method* either:
  - *Standard create* - provide detailed changes in configuration options.
  - *Easy create* - use recommended best-practice configurations.
  
- Next, choose *Configuration*:
  - Choose *Engine options*:
    - Postgres
    - MySQL
    - MariaDB
    - Oracle
    - Microsoft SQL Server
    - Aurora (AWS Proprietary database) ✔️
  - Select *Aurora Edition*.
    - Amazon Aurora MySQL-compatible edition
    - Amazon Aurora PostgreSQL-compatible edition
  - Select *Capacity type*.
    - Provisioned (user-provisioned and managed the server instance sizes)
    - Serverless (user specifies the minimum and maximum amount of resources needed, and Aurora scales the capacity based on database load)
  - Within *Replication features*, you can either select *single-master* or *multi-master* (when continuous writer availability is required).
  
- Next, within *Templates*, choose your use case:
  - Production
  - Dev/Test
  - Free Tier
  
- Next, within *Settings*:
  - Provide *DB instance identifier*
  - Under *Credential settings*, 
    - Give a master username.
    - Give a master password.
  
- Next, within *DB instance class*, choose:
  - Memory optimized classes (r classes)
  - Burstable classes (t classes)
  
- Next, within *Availability & duration*, select for Multi-AZ deployment, it can either be:
  - *Do not create a standby instance*. (**even here your database layer is replicated across the multi AZ.**)
  - *Create an Aurora Replica or Reader node in a different AZ (recommended for scaled availability).* 
  
- Next, within *Connectivity*:
  - Provide *VPC*.
  - Provide *Subnet group*.
  - Select whether or not you want to provide *Public access*.
  - Provide *VPC security group*.
  - Provide *AZ* (preference).
  
- Next, within *Database authentication*, select Database authentication options:
  - Password authentication (*Authenticates using database passwords*)
  - Password and IAM Database authentication (*Authentication using database password and user credentials through AWS IAM roles*)
  
- Next, within *Additional configuration*:
  - We have *Database options*, for providing:
    - *Initial database name*
    - *DB parameter group*
    - *Option group*
    - *Failover priority*
  - We have *Backup*:
    - Select *Enable automatic backups*.
    - Provide *Backup retention period.*
    - Select *Backup window.*
    - Enable *Cop tags to snapshots.*
  - We have *Encryption*:
    - Provide *AWS KMS Key*.
  - We have *Backtrack*:
    - Select/leave *Enable Backtrack*.
  
  - We have *Monitoring*:
    - Select *Enable Enhanced monitoring*.
    - Select *Granularity* period.
    - Select *Monitoring Role*.
  - For *Log exports*, select favored log types to publish to Amazon CloudWatch logs.
    - Audit log
    - Error log
    - General log
    - Slow query log
  - For *Maintenance*,
    - Select *enable auto minor version upgrade*.
    - Select Maintenance window.
  - For *Deletion protection*, select *enable deletion protection*.
  
- Create RDS.

Once created,

1. If go to Connectivity & security > Endpoints, we get the endpoints we can use to within our applications.
2. We can also perform the following actions:
   - Add AWS Region
   - Add reader
   - Create cross-region read replica
   - Create clone
   - Restore to point in time
   - Add replica auto scaling

### Advanced Concepts

#### Aurora Replicas - Autoscaling

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/06-aws-fundamentals-rds_aurora_elasticache/assets/aurora-rds-autoscaling.svg)

#### Aurora - Custom Endpoints

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/06-aws-fundamentals-rds_aurora_elasticache/assets/aurora-custom-endpoints.svg)

- Define a subset of Aurora instances as a custom endpoint.
- Example use case, run analytical queries on specific replicas.
- The reader endpoint is generally not used after defining custom endpoints.

#### Aurora Serverless

- Automated database instantiation and auto-scaling based on actual usage.
- Good for infrequent, intermittent or unpredictable workloads
- No capacity planning needed.
- Pay per second, can be more cost-effective.

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/06-aws-fundamentals-rds_aurora_elasticache/assets/aurora-serverless.svg)

#### Aurora - Multi master

- In case you want immediate failover for write node (HA)
- Every node does R/W - vs promoting a RR as the new master

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/06-aws-fundamentals-rds_aurora_elasticache/assets/aurora-multi-master.svg)

#### Global Aurora 

- Aurora cross region read replicas:
  - useful for disaster recovery
  - simple to put in place
- Aurora global database (recommended):
  - 1 primary region (read/write)
  - up to 5 secondary (read-only) regions, replication lag < 1 second
  - up to 16  read replicas per secondary region
  - helps for decreasing latency
  - promoting another region (for disaster recovery) has an RTO < 1 minute

#### Aurora ML

- Enables you to add ML-based predictions to your applications via SQL
- Simple, optimized and secure integration between Aurora and AWS ML services.
- Supported services
  - Amazon SageMaker (use with any ML model)
  - Amazon Comprehend (for sentiment analysis)
- You don't need to have ML experience
- Use case: 
  - fraud detection
  - ads targeting
  - sentiment analysis
  - product recommendations

## ElastiCache 

- ElastiCache is to get managed Redis or Memcached
- AWS takes care of OS maintenance/patching, optimizations, setup, configuration, monitoring, failure recovery and backups.

> Note: <span style="color:red">**using ElasticCache involves heavy application code changes**</span>

#### Architecture 

##### DB Cache

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/06-aws-fundamentals-rds_aurora_elasticache/assets/elasticcache-architecture-db-cache.svg)

##### User Session Store

- User logs into any of the application
- The application writes the session data into ElastiCache
- The user hits another instance of our application
- The instance retrieves the data and the user automatically gets logged in.

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/06-aws-fundamentals-rds_aurora_elasticache/assets/elasticcache-architecture-user-session-store.svg)

#### ElasticCache - Redis vs Memcached

| Redis                                                        | Memcached                                          |
| ------------------------------------------------------------ | -------------------------------------------------- |
| **Multi AZ** with auto failover                              | Multi-node for partitioning of data (**sharding**) |
| **Read replicas** to scale reads and have **high availability** | **No high availability** (replication)             |
| Data durability using AOF persistence                        | **Non persistence**                                |
| **Backup and restore features**                              | **No backup and restore**                          |
|                                                              | Multi-threaded architecture                        |





**Hands-On**

## Questions