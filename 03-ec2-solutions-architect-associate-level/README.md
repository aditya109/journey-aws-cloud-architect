Table of Contents

- [EC2 - Solutions Architect Associate Level](#ec2---solutions-architect-associate-level)
  * [Private vs Public vs Elastic IP](#private-vs-public-vs-elastic-ip)
  * [EC2 Placement Groups](#ec2-placement-groups)
  * [Elastic Network Interfaces (ENI) - Overview](#elastic-network-interfaces--eni----overview)
    + [Hands-On](#hands-on)
  * [ENI - Extra Reading](#eni---extra-reading)
  * [EC2 Hibernate](#ec2-hibernate)
  * [EC2 Advanced concepts](#ec2-advanced-concepts)
    + [EC2 Nitro](#ec2-nitro)
    + [EC2 - Understanding CPU](#ec2---understanding-cpu)
    + [EC2 - Capacity Reservations](#ec2---capacity-reservations)

# EC2 - Solutions Architect Associate Level

## Private vs Public vs Elastic IP

- Networking has two sorts of IPs. IPv4 and IPv6:
  
  - IPv4: `1.160.10.240` (allows for 3.7 billion different addresses in the public space)
  
  - IPv6: `3ff3:1900:4545:3:200:f8ff:fe21:67cf`.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/03-ec2-solutions-architect-associate-level/assets/private-vs-public-ip.svg) 

| Private IP                                                                                                  | Public IP                                                                           | Elastic IPs                                                                                                                                           |
| ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| Private IP means the machine can only be identified on a private network only.                              | It means the machine can be identified on the internet (WWW).                       | When you stop and then start an EC2, it can change its public IP. If you need to have a fixed public IP for your instance, you need an Elastic IP.    |
| The IP must be unique across the private network, but two different private networks can have the same IPs. | Must be unique across the whole wen (not two machines can have the same public IP). | An Elastic IP is a public IPv4 IP you own as long as you don' delete it.                                                                              |
| Machines connect to WWW using a NAT + internet gateway (a proxy).                                           | Can be geo-located easily.                                                          | You can attach it to one instance at a time.                                                                                                          |
| Only a specified range of IPs can be used as private IPs.                                                   |                                                                                     | With an Elastic IP address, you can mask the failure of an instance or software by rapidly remapping the address to another instance in your account. |
|                                                                                                             |                                                                                     | You can only have 5 Elastic IP in your account (you can ask AWS to increase that).                                                                    |

> Overall, **<span style="color:orange">try to avoid using Elastic IP</span>**, they often reflect poor architectural decisions, instead:
> 
> - Use a random public IP and register a DNS name to it, or;
> 
> - Use a Load Balancer and don't use a public IP. 

By default, EC2 machine comes with:

- A private IP for the internal AWS network.

- A public IP, for the WWW.

When we are doing SSH into our EC2 machines:

- We can't use a private IP, because we are not in the same network.

- We can only use the public IP.

If the machine is stopped and then started, the public IP can change.

To use an Elastic IP, go to `Network and Security` > `Elastic IPs` > Create an Elastic IP, now `Actions`> `Associate Elastic IP Address`.

> <span style="color:crimson">On creating an Elastic IP, you'd be charged unless you don't have it associated to an EC2 instance.</span> 
> So associate it to an EC2 instance as soon as possible.
> 
> <span style="color:orange">Or if not required go to `Actions` > `Release Elastic IP Address` to release the unused IP.</span>

Now on stopping our instance, we can observe that the Public IP does not go away, it is retained as an Elastic IP is tied to the instance.

## EC2 Placement Groups

Sometimes you'd want control over the EC2 Instance placement strategy. That strategy can be defined using placement groups. When you create a placement group, you can specifiy one of the following strategies for the group:

- *Cluster* - clusters instances into a low-latency group in a single Availability Zone.)
  ![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/03-ec2-solutions-architect-associate-level/assets/cluster-placement-group.svg)

- *Spread* - spreads instances across underlying hardware (max 7 instances per group per AZ) - critical applications.)
  ![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/03-ec2-solutions-architect-associate-level/assets/spread-placement-group.svg)

- *Partition* - spreads instances across many different partitions (which rely on different sets of racks) within an AZ. Scales to 100s of Ec2 instances per group. (Hadoop, Cassandra, Kafka))
  ![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/03-ec2-solutions-architect-associate-level/assets/partition-placement-group.svg)
  
  Each partition is a rack (upto 7 partitions per AZ).

|                | Cluster                                                                                                                  | Spread                                                                                                                                               | Partition                                                                                                                                                                                                                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Characteristic | Same rack, same AZ<br/>(low latency, 10Gbps network)                                                                     | Different rack, different AZ                                                                                                                         | Can span across multiple AZs in the same region.                                                                                                                                                                                                                                          |
| Pros           | Great network (10Gbps bandwidth network between instances)                                                               | Can span across AZs<br/>Reduced risk of simultaneous failure<br/>Ec2 Instances are on different physical hardware.                                   | Upto to 100s of EC2 instances.<br/>The instances in a partition do not share racks with the instances in the other partitions.<br/>A partition failure can affect many EC2 but won't affect other partitions.<br/><br/>EC2 instances get access to the partition information as metadata. |
| Cons           | If the rack fails, all instances fails at the same time.                                                                 | Limited to 7 instances per AZ per placement group.                                                                                                   | Limited to 7 partitions per AZ                                                                                                                                                                                                                                                            |
| Usecase        | Big Data job that needs to complete fast.<br/>Applications that needs extremely low latency and high network throughput. | Application that needs to maximize high availablility.<br/>Critical applications where each instances must be isolated from failure from each other. | HDFS, HBase, Cassandra, Kafka                                                                                                                                                                                                                                                             |

To create a placement group, 

1. We go to `Networking and Security` > `Placement Groups`.

2. Provide a Name, `Placement Strategy` (Cluster/Spread/Partition) and Tags (if required).

Now while creating an EC2 Instance, within `Configure Instance`, see `Placement Group`

> You can leave the created `Placement Groups`, you will not be charged any money for idle PGs.

## Elastic Network Interfaces (ENI) - Overview

They are logical component in a VPC that represents a <span style="color:cyan">*virtual network card*</span>.
![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/03-ec2-solutions-architect-associate-level/assets/eni.svg)

- The ENI can have the following attributes:
  
  - Primary private IPv4, one or more secondary IPv4
  
  - One Elastic IP (IPv4) per private IPv4
  
  - One Public IP (IPv4)
  
  - One or more security groups
  
  - A MAC Address

- You can create ENI independently and attach them on the fly (move them) on EC2 instances for failover.

- <span style="color:orange">Bound to a specific AZ.</span>

### Hands-On

Go to `Networking and Security` > `Network Interfaces`.

To create a new ENI, 

1. We give a name.

2. Assign a `Subnet` (has to be in the same AZ as the EC2 instance).

3. Get a private IPv4 address.

4. Attach a Security Group. 

> Elastic Fabric Adapter is a network interface for Amazon EC2 instances that enables customers to run applications requiring high levels of inter-node communications at scale on AWS. *Look into Elastic Fabric Adapter. [Elastic Fabric Adapter — Amazon Web Services](https://aws.amazon.com/hpc/efa/)*
> 
> Its custom-built OS bypass hardware interface enhances the performance of inter-instance communications, which is critical to scaling these applications. 

Now we can attach it to our instance. (From `Actions`)

> Main advantage here is drastic reduction in network failover, meaning we can switch the same ENI between multiple instances.

## ENI - Extra Reading

[New – Elastic Network Interfaces in the Virtual Private Cloud | AWS News Blog](https://aws.amazon.com/blogs/aws/new-elastic-network-interfaces-in-the-virtual-private-cloud/)

## EC2 Hibernate

We know we can stop, terminate instances:

- `Stop`: the data on disk (EBS) is kept intact in the next start.

- `Terminate`: any EBS volumes (root) also set-up to be destroyed is lost.

On start, the following happens:

- First start: the OS boots & the EC2 UserData script is run.

- Following starts: the OS boots up.

- The your application starts, caches get warmed up and that can take time.

**To resolve this issue, we use EC2 Hibernate.**

- The in-memory (RAM) state is preserved.

- The instance boot is much faster ! (the OS is not stopped / restarted)
  Actually the RAM state us written to a file in the root EBS volume.

- The root EBS volume must be encrypted.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/03-ec2-solutions-architect-associate-level/assets/ec2-hibernate.svg)

- Supported instance families - C3, C4, C5, M3, M4, M5, R3, R4 and R5.  

- Instance RAM size - must be less than 150 GB.

- Instance size - not supported for bare metal instances.

- AMI: Amazon Linux 2, Linux AMI, Ubuntu ... & WIndows

- Root Volume: must be EBS, encrypted, <span style="color:orange">not instance store and large</span>

- Available for On-Demand and Reserved Instances.

- An instance cannot be hibernated more than 60 days.

**Hands On**

To enable EC2-Hibernate, while creating an EC2 instance, in `Configure Instance`,

- Look for `Stop -Hibernate Behaviour` and enable `hibernatation as an additional stop behaviour`. 
  
  > We need to make sure that the root volume has more storage than RAM on the instance and that the encryption on root volume is `Enabled`. 

## EC2 Advanced concepts

### EC2 Nitro

- Underlying platform for the next generation of EC2 instances.

- Better and new virtualization technology.

- Allows for better performance:
  
  - Better networking options (enhanced networking, HPC, IPv6)
  
  - Higher Speed EBS (Nitro is necessary for 64k EBS IOPS (max 32k on non-Nitro instances))

- Better underlying security.

- Instance type example: Virtualized (A1, C5, M5a, etc.,), Bare Metal (a1.metal, c5.metal, etc.)

### EC2 - Understanding CPU

- Multiple threads can run on one CPU (multithreading)

- Each thread is represented as a virtual CPU (vCPU)

For example, for `m5.2xlarge`, we have 4 CPU, 8 threads (***8 vCPU***)

**What is the point here ? 🤔**

EC2 instances come with a combination of RAM and vCPU. But in some cases, you may want to chance the vCPU options:

- `#` of CPU cores: you can decrease it (helpful if you need high RAM and low number of CPU) - to decrease licensing costs.

- `#` of threads per core: disable multithreading to 1 thread per CPU 0 helpful for HPC workloads.

<span style="color:orange">Only specified during instance launch</span>.
In `CPU Options` > check `Specify CPU options` and there you can do the adjustment.

### EC2 - Capacity Reservations

- Capacity Reservations ensure you have EC2 Capacity when needed.

- Manual or planned end-date for the reservation.

- No need for 1 or 3-year commitment.

- Capacity access is immediate, you get billed as soon as it starts.

You need specify the following for creating such instances:

- The AZ in which to reserve the capacity (only one)

- The # of instances for which to reserve capacity

- The instance attributes, including the instance type, tenancy and platform/OS.

> Combine with Reserved Instances and Savings Plans to do cost saving.
