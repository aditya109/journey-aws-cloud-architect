**Table of Contents:**

- [EC2 Fundamentals](#ec2-fundamentals)
  * [EC2 Basics](#ec2-basics)
    + [EC2 sizing & configuration options](#ec2-sizing---configuration-options)
    + [EC2 User Data](#ec2-user-data)
    + [Create EC2 instance with EC2 user data to have a website](#create-ec2-instance-with-ec2-user-data-to-have-a-website)
  * [EC2 instance type basics](#ec2-instance-type-basics)
    + [EC2 - General Purpose](#ec2---general-purpose)
    + [EC2 - Compute Optimized](#ec2---compute-optimized)
    + [EC2 - Memory Optimized](#ec2---memory-optimized)
    + [EC2 - Storage Optimized](#ec2---storage-optimized)
  * [Security Groups and classics ports overview](#security-groups-and-classics-ports-overview)
    + [Introduction to Security Groups](#introduction-to-security-groups)
    + [Security Groups - A deep dive](#security-groups---a-deep-dive)
      - [Security Groups Diagram](#security-groups-diagram)
    + [Referencing other security groups](#referencing-other-security-groups)
    + [Classic Ports to know](#classic-ports-to-know)
  * [## SSH Overview](#---ssh-overview)
  * [SSH Troubleshooting](#ssh-troubleshooting)
  * [EC2 Instance connect](#ec2-instance-connect)
  * [EC2 Instance roles demo](#ec2-instance-roles-demo)
  * [EC2 Instance launch types](#ec2-instance-launch-types)
    + [EC2 Instances Purchasing Options](#ec2-instances-purchasing-options)
      - [EC2 On Demand](#ec2-on-demand)
      - [EC2 Reserved Instances](#ec2-reserved-instances)
        * [Convertible Reserved Instance](#convertible-reserved-instance)
        * [Scheduled Reserved Instances](#scheduled-reserved-instances)
      - [EC2 Spot Instances](#ec2-spot-instances)
      - [EC2 Dedicated Hosts](#ec2-dedicated-hosts)
        * [EC2 Dedicated Instances](#ec2-dedicated-instances)
    + [Purchasing options - Hotel analogy](#purchasing-options---hotel-analogy)

# EC2 Fundamentals

**Caution:** [AWS Budget setup](https://www.udemy.com/course/aws-certified-developer-associate-dva-c01/learn/lecture/26100826?start=30#overview)

## EC2 Basics

- EC2 or Elastic Compute Cloud = Infrastructure as a Service
- It consists in the capability of :
  - Renting virtual machines (EC2)
  - Storing data on virtual drives (EBS)
  - Distributing load across machines (ELB)
  - Scaling the services using an auto-scaling group (ASG)

### EC2 sizing & configuration options

- OS: Linux, Windows or MacOS
- How much compute power & cores (CPU)
- How much random-access memory (RAM)
- How much storage space:
  - Network-attached (EBS & EFS)
  - Hardware (EC2 Instance store)
- Network card: speed of the card, public IP address
- Firewall rules: **security group**
- Bootstrap script (configure at first launch): EC2 User Data

### EC2 User Data

- It is possible to bootstrap our instances using an EC2 User data script.
- `bootstrapping` means launching commands when a machine starts.
- The script is only run once at the instance first starts.
- EC2 user data is used to automate boot tasks such as:
  - Installing updates
  - Installing software
  - Downloading of common files from the internet
- EC2 user data script runs with the root user.

### Create EC2 instance with EC2 user data to have a website

1. We start off with an AMI (Amazon Machine Image, a template that contains the software configuration (OS, app server, and applications) required to launch your instance).
2. Then we choose Instance Type and configure instance.
3. Configure storage and add tags.
4. We then configure Security Groups (A Security Group is a set of firewall rules that control the traffic for your instance)

Few things which I noticed:

- When we start `Start` an instance, it gets a public and a private IP address.
- We can `Start`, `Stop` or `Terminate` an instance.
- Whenever we restart an instance, its private IP remains the same, but the public IP changes.

## EC2 instance type basics

An instance is a virtual server in the cloud. Its configuration at launch is a copy of the AMI that you specified when you launched the instance.

We have 7 different types of instances:

1. General purpose
2. Compute optimized
3. Memory optimized
4. Accelerated computing
5. Storage optimized
6. Instance features
7. Measuring instance performance

AWS has the following naming conventions:

```bash
m5.2xlarge
```

- `m` - instance class
- `5` - generation (AWS improves them over time)
- `2xlarge` - size within the instance class

Use [EC2Instances.info](https://instances.vantage.sh/) for easy Amazon EC2 instance comparison.

### EC2 - General Purpose

General purpose instances provide a balance of *compute*, *memory* and *networking* resources, and can be used for a variety of diverse workloads. 
They are ideal for applications that use resources in equal proportions such as web servers and code repositories.

### EC2 - Compute Optimized

Compute optimized instances are ideal for compute bound applications that benefit from high performance processors. 
They are ideal for batch processing workloads, media transcoding, high performance web servers, high performance computing (HPC), scientific modeling , dedicated gaming servers and ad server engines, machine learning inference and other compute intensive applications.  

### EC2 - Memory Optimized

Memory optimized instances are designed to deliver fast performance for workloads that process large data sets in memory.
It is ideal for memory-intensive applications such as :

1. high-performance, relational/non-relational databases, 
2. distributed web scale cache stores (in-memory caches),
3. in-memory databases optimized for BI (business intelligence),
4. applications performing real-time processing of big unstructured data.

### EC2 - Storage Optimized

Storage optimized for storage-intensive tasks that require high, sequential read and write access to large datasets on local storage. They are optimized to deliver tens of thousands of low-latency, random I/O operations per seconds (IOPS) to applications.
Usecases:

1. High frequence online transaction processing (OLTP) systems,
2. Relational & NoSQL databases,
3. Cache for in-memory databases,
4. Data warehousing applications,
5. Distributed file systems 

## Security Groups and classics ports overview

### Introduction to Security Groups

Security groups are the fundamental of network security in AWS.
They control how traffic is allowed into or out of our EC2 instances.
Security groups only contain **<span style="color:green">allow</span>** rules. They can reference by IP or by security group.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/e036c1e99108e5d21bc2f41b22b52f174c326970/02-ec2-fundamentals/assets/security-group-introduction.svg)

### Security Groups - A deep dive

Security groups are acting as a *firewall* on the EC2 instances. 
They regulate:

1. access to ports 
2. authorized IP ranges - IPv4 and IPv6
3. control of inbound network (from other to the instance)
4. control of outbound network (from the instance to other)

| Type   | Protocol | Port Range | Source            | Description    |
| ------ | -------- | ---------- | ----------------- | -------------- |
| HTTP   | TCP      | 80         | 0.0.0.0/0         | test http page |
| SSH    | TCP      | 22         | 122.149.196.85/32 |                |
| Custom | TCP      | 4567       | 0.0.0.0/0         | java app       |

#### Security Groups Diagram

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/02-ec2-fundamentals/assets/security-group-diagram.svg)

**Good to know things about Security Groups**

- Can be attached to multiple instances.

- Locked down to a region / VPC combination.

- Does live `outside` the EC2 - if traffic is blocked the EC2 won't see it.
  
  > Its' good to maintain one separate security group for SSH access.

- If your application is not accessible (timeout), then its' a security group issue.

- If your application gives a *connection refused* error, then its' an application error or its' not launched.

- All inbound traffic is <span style="color:orange"> blocked </span> by default.

- All outbound traffic is <span style="color:green">authorised</span> by default.

### Referencing other security groups

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/02-ec2-fundamentals/assets/referencing-other-security-groups.svg)

### Classic Ports to know

| Port | Usecase                              | Description                    |
| ---- | ------------------------------------ | ------------------------------ |
| 22   | SSH (Secure Shell)                   | log into a linux instance      |
| 21   | FTP (File Transfer Protocol)         | upload files into a file share |
| 22   | SFTP (Secure File Transfer Protocol) | upload files using SSH         |
| 80   | HTTP                                 | access unsecured websites      |
| 443  | HTTPS                                | access secured websites        |
| 3389 | RDP (Remote Desktop Protocol)        | log into a Windows instance    |

## ## SSH Overview

We can connect to EC2 via:

1. `Putty`

2. `ssh`

3. EC2 Instance Connect

On doing the following, when first performing SSH into the instance:

```bash
ssh -i EC2Tutorial.pem ec2-user@35.180.100.144
```

we get the following error:

```bash
Permission 0644 for 'EC2Tutorial.pem' are too open.
It is required that your private key files are NOT accessible by others
This private key will be ignored.
Load key "EC2Tutorial.pem": bad permissions
```

To fix this:

1. On Linux

```bash
chmod 0400 EC2Tutorial.pem #on Linux
ssh -i EC2Tutorial.pem ec2-user@35.180.100.144
```

2. On Windows
- Go to `Properties` of EC2Tutorial > `Security` tab > `Advanced` section.

- Change owner of the key file to yourself.

- Click on `Disable Inheritance`.

- Remove all other users, except yourself.

- Click on `Apply`.

- Go back, ensure that the only user with full permissions of the file is you, yourself.

- Try doing SSH on the file.

```bash
ssh -i EC2Tutorial.pem ec2-user@35.180.100.144
```

## SSH Troubleshooting

1. **There is a connection timeout.**
   
   This is a security group issue. Any timeout is related to security groups or a firewall. Ensure your security group looks like this and correctly assigned to your EC2 instance.

2. **There is still a connection timeout issue.**
   A corporate or personal firewall is blocking the connection. *Try using EC2 Instance Connect.*

3. **There's a connection refused.**
   This means the instance is reachable, but no SSH utility is running on the instance.
   
   1. Try restarting the instance.

4. `Permission denied (publickey,gssapi-keyex,gssapi-with-mic)`
   
   This means either two things:
   
   - You are using the wrong security key or not using a security key. Please look at your EC2 instance configuration to make sure you assigned the correct key to it.
   
   - You are using the wrong user. Make sure you have started an **Amazon Linux 2 EC2 Instance**, and make sure you're using the user *ec2-user*. 
     This is something you specify when doing `ec2-user@<public-ip>` (ex: `ec2-user@35.180.242.162`) in your SSH command or your Putty configuration.

## EC2 Instance connect

You need to have SSH port open in inbound rules to connect to available EC2 instances.

## EC2 Instance roles demo

> It is always a bad idea to store aws credentials like `Access Key` and `Secret Access Key` on the EC2 as that could enable anyone having access to our EC2 instance and steal those credentials.

Use <span style="color:orange">**IAM roles**</span>.

Click on the EC2 instance > `Actions` > `Security` > `Modify IAM Role` > Choose an IAM Role.

## EC2 Instance launch types

### EC2 Instances Purchasing Options

- On-demand instances: short workload, predictable pricing

- Reserved: (MINIMUM 1 year)
  
  - Reserved instances: long workloads
  
  - Convertible Reserved instances: long workloads with flexible instances.
  
  - Scheduled Reserved instances: *e.g.,* every Thursday between 3 and 6 pm.

- Spot Instances: short workloads, cheap, can loose instance (less reliable)

- Dedicated Instances: book an entire physical server, control instance placement 

#### EC2 On Demand

- Pay for what you use:
  
  - Linux or Windows - billing per second, after the first minute
  
  - All other operating systems - billing per hour

- Has the highest cost but no upfront payment

- No long-term commitment
  
  > Recommended for short-term and un-interrupted workloads, where you can't predict how the application will behave.

#### EC2 Reserved Instances

- Up to 75% discount compared to `On-demand`.

- Reservation period: 1 year = + discount | 3 years = +++ discount.

- Purchasing options: no upfront | partial upfront = + | All upfront = ++ discount.

- Reserve a specific instance type.

- Recommended for steady-state usage applications (think database).

##### Convertible Reserved Instance

- can change the EC2 Instance type.

- upto 54% discount.

##### Scheduled Reserved Instances

- launch within time window you reserve.

- when you require a fraction of day/week/month.

- still commitment over 1 to 3 years.

#### EC2 Spot Instances

- Can get a discount of upto 90% compared to `On-demand`.

- Instances that you can *lose* at any point of time if your max price is less than the current spot price.

- The MOST cost-efficient isntances in AWS

> Useful for workloads that are resilient to failure.
> 
> - Batch jobs
> 
> - Data analysis
> 
> - Image processing
> 
> - Any distributed workloads
> 
> - Workloads with a flexible start and end time
> 
> <span style="color:orange">Not suitable for critical jobs or databases.</span>

#### EC2 Dedicated Hosts

- An Amazon EC2 Dedicated Host is a physical server with EC2 instance capacity  fully dedicated to your use. Dedicated hosts can help you address **compilance requirements** and reduce costs by allowing you to **use your existing server-bound software license**.

- Allocated for your account for a 3-year period reservation

- More expensive

> Useful for software that have complicated licensing model (BYOL - Bring Your Own License), or for companies that have strong regulatory or compilance needs.

##### EC2 Dedicated Instances

- Instances running on hardware that's dedicated to you.

- May share hardware with other instances in same account.

- No control over instance placement (can move hardware after Start/Stop).

| Characteristic                                          | Dedicated Instances | Dedicated Hosts |
| ------------------------------------------------------- | ------------------- | --------------- |
| Enables the use of dedicated physical servers           | ✅                   | ✅               |
| Per instance billing (subject to a $$2$ per region fee) | ✅                   |                 |
| Per host billing                                        |                     | ✅               |
| Visibility of sockets, cores, host ID                   |                     | ✅               |
| Affinity between a host and instance                    |                     | ✅               |
| Targeted instance placement                             |                     | ✅               |
| Automatic instance placement                            | ✅                   | ✅               |
| Add capacity using an allocation request                | ✅                   | ✅               |

### Purchasing options - Hotel analogy

- **On demand**: coming and staying in hotel whenever we like, we pay the full price.

- **Reserved**: like planning ahead and if we plan to stay for a long time, we may get a good discount.

- **Spot instances**: the hotel allows people to bid for the empty rooms and the highest bidder keeps the rooms. You can get kicked out at any time.

- **Dedicated Hosts**: We book an entire building of the hotel.

## Spot Instances & Spot Fleet

- Can get a discount of upto 90% compared to `On-demand`.

- Define **max spot price** and get the instance while **current spot price < max**.
  
  - The hourly spot price varies based on offer and capacity.
  
  - If the current spot price > max price you can choost to **stop** or **terminate** your instance with a 2-minute grace period.

- Other strategy: **Spot Block**
  
  - *block* spot instance during a specified time frame (1 to 6 hours) without interruptions.
  
  - In rare situations, the instance may be reclaimed.

> Used for batch jobs, data analysis or workloads that are resilient to failures.
> <span style="color:orange">Not suitable for critical jobs or databases.</span>

<strong>How to terminate Spot instances ?</strong>

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/02-ec2-fundamentals/assets/terminate-spot-instances.svg)

You can only cancel Spot instance requests that are **open, active or disabled**.

<span style="color:crimson">**Cancelling a Spot Instance does not terminate instances.**</span>

<span style="color:orange">You must first cancel a Spot Request, and then terminate the associated Spot instances.</span>

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/02-ec2-fundamentals/assets/difference-between-one-time-and-persistent.svg)

### Spot Fleets

- Spot Fleets = set of Spot Instances + (optional) On-Demand instances.

- The Spot Fleet wll try to meet the target capacity with price constraints.
  
  - Define possible launch pools: instance type (m5.large), OS, Availability Zones (AZ)
  
  - Can have multiple launch pools, so that the fleet can choose.
  
  - Spot Fleet stops launch instances when reaching capacity or max cost.

- Strategies to allocate Spot Instances:
  
  - **lowestPrice**: from the pool with the lowest price (cost optimization, short workload)
  
  - **diversified**: distributed across all pools (great for availability, long workloads)
  
  - **capacityOptimized**: pool with the optimal capacity for the number of instances.

- <span style="color:limegreen">**Spot Fleets allow us to automatically request Spot Instances with the lowest price.**</span>
