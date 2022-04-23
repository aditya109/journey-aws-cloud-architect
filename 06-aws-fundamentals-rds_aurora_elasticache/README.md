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
  - 

### RDS Encryption + Security

## Amazon Aurora

**Hands-On**

### Advanced Concepts

## ElastiCache Hands-On

## Questions