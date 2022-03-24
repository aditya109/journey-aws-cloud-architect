Table of Contents

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

- *Spread* - spreads instances across underlying hardware (max 7 instances per group per AZ) - critical applications.

- *Partition* - spreads instances across many different partitions (which rely on different sets of racks) within an AZ. Scales to 100s of Ec2 instances per group. (Hadoop, Cassandra, Kafka)

|                | Cluster                                                                                                                       | Spread | Partition |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------ | --------- |
| Characteristic | Same rack, same AZ<br/>(low latency, 10Gbps network)                                                                          |        |           |
| Pros           | Great network (10Gbps bandwidth network between instances)                                                                    |        |           |
| Cons           | If the rack fails, all instances fails at the same time.                                                                      |        |           |
| Usecase        | Big Data job that needs to complete fast.<br/><br/>Applications that needs extremely low latency and high network throughput. |        |           |

## Elastic Network Interfaces (ENI) - Overview

## ENI - Extra Reading

## EC2 Hibernate

## EC2 Advanced concepts
