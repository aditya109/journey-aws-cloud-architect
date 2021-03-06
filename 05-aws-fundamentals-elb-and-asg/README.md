# High Availability and Scalability: ELB & ASG

## Table of Contents

- [High Availability, Scalability and Load Balancing](#high-availability--scalability-and-load-balancing)
- [ELB Overview](#elb-overview)
  * [Health Checks](#health-checks)
    + [Types of LBs](#types-of-lbs)
    + [LB Security Groups](#lb-security-groups)
  * [Sticky Sessions](#sticky-sessions)
    + [Session Affinity](#session-affinity)
    + [Cookie Names](#cookie-names)
  * [Cross Zone Load Balancing](#cross-zone-load-balancing)
    + [Cross Zone Load Balancing Feature Availability on Load Balancer](#cross-zone-load-balancing-feature-availability-on-load-balancer)
  * [SSL Certificates](#ssl-certificates)
    + [SSL - Server Name Indication](#ssl---server-name-indication)
    + [ELB - SSL Certificates](#elb---ssl-certificates)
  * [Connection Draining](#connection-draining)
- [CLB (v1)](#clb--v1-)
- [Application Load Balancer (v2)](#application-load-balancer--v2-)
  * [Target Group Possiblities for ALB](#target-group-possiblities-for-alb)
  * [Query strings/parameters routing](#query-strings-parameters-routing)
- [Network Load Balancer (v2)](#network-load-balancer--v2-)
  * [Target Group Possiblities for NLB](#target-group-possiblities-for-nlb)
- [Gateway Load Balancer](#gateway-load-balancer)
  * [Target Group Possiblities for NLB](#target-group-possiblities-for-nlb-1)
  * [Auto Scaling Group Overview](#auto-scaling-group-overview)
    + [Attributes](#attributes)
    + [Auto scaling alarms](#auto-scaling-alarms)
    + [Auto Scaling New Rules](#auto-scaling-new-rules)
    + [Auto Scaling Custom Metric](#auto-scaling-custom-metric)
  * [Scaling Policies](#scaling-policies)
    + [Good metrics to scale on](#good-metrics-to-scale-on)
    + [Scaling Cooldowns](#scaling-cooldowns)
- [Questions (important topics):](#questions--important-topics--)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## High Availability, Scalability and Load Balancing

Scalability means than an application/system can handle greater loads by adapting.
There are two kinds of scalability:

1. Vertical scalability;

2. Horizontal scalability (=elasticity).

>  *Scalability is linked but linked to high availability.*

High availability means running your application/system in at least 2 data centers (AZ).
The high availability can be:

1. Passive (for RDS Multi AZ for example);

2. Active (for horizontal scaling).

> The goal of high availability is to survive a data center loss. 

Load balancers are servers that forward traffice to multiple servers downstream.

*Why use a load balancer?*

- Spread load across multiple downstream instances.

- Expose a single point of access (DNS) to your application.

- Seamlessly handle failures of downstream instances.

- Do regular health checks to your instances.

- Provide SSL termination (HTTPS) for your websites.

- Enforce stickiness with cookies.

- High availablility across zones.

- Separate public traffic from private traffic.

## ELB Overview

An Elastic Load Balancer is 

- a <span style="color:orange">managed load balancer</span>,
  
  - AWS guarantees that it will be working
  
  - AWS takes care of upgrades, maintenance, high availability
  
  - AWS provides only a few configuration knobs

- cheap to setup your own load balancer but it will be a lot more effort on your end

- integrated with many AWS offerings/services
  
  - EC2, EC2 Auto scaling groups, ECS
  
  - AWS Certificate Manager (ACM), CloudWatch
  
  - Route 53, AWS WAF, AWS Global Accelerator

### Health Checks

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/health-check.svg)

- Health checks are crucial for Load Balancers

- They enable the lb to know if the instances it forwards traffic to are available to reply to requests.

- The health check is done on a port and a route (/health is common)

- If the reponse is not 200 (OK), then the instance is unhealthy

#### Types of LBs

AWS has 4 kinds of managed Load Balancers:

| LB type              | Year of Launch | Supported Protocol                                | Remarks             |
| -------------------- | -------------- | ------------------------------------------------- | ------------------- |
| Classic LB - CLB     | 2009           | HTTP, HTTPS, TCP, SSL (secure TCP)                | v1 - old generation |
| Application LB - ALB | 2016           | HTTP, HTTPS, WebSocket                            | v2 - new generation |
| Network LB - NLB     | 2017           | TCP, TLS (secure TCP), UDP                        | v2 - new generation |
| Gateway LB - GWLB    | 2020           | Operates at layer 3 (Network layer) - IP Protocol | latest              |

> It is recommended to use the newer generation lbs as they provide more features.
> 
> Some lbs can be setup as internal (private) or external (public) ELBs.

#### LB Security Groups

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/lb-security-group.svg)

|                                               | LB Security Group                             | Application Security Group (allow traffic only from lb) |
| --------------------------------------------- | --------------------------------------------- | ------------------------------------------------------- |
| *type-protocol-port range-source-description* | HTTP-TCP-80-0.0.0.0/0-allow HTTP from an....  | HTTP-TCP-80-`sg-ID1`-allow HTTP from an....             |
| *type-protocol-port range-source-description* | HTTP-TCP-443-0.0.0.0/0-allow HTTP from an.... |                                                         |

### Sticky Sessions

#### Session Affinity

- It is possible to implement stickiness so that the same client is always redirected to the same instance behind a load balancer, this works for CLB and ALB.

- The *cookie* used for stickiness has an expiration date you control.

- Pros: Ensure the user doesn't lose his session data. (usecase possibility as well) 

- Cons: Enabling stickiness may bring imbalance to the load over the backend EC2 instances.

#### Cookie Names

- **Application-based** cookies.
  
  - It could be:
    
    1. a *custom cookie*, which is,
       
       - Generated by the target,
       
       - Can include any custom attributes required by the application,
       
       - Cookie name must be specified individually for each target group,
       
       - Don't use <span style="color:orange">AWSALB</span>, <span style="color:orange">AWSALBAPP</span> or <span style="color:orange">AWSALBTG</span> (reserved for use by the ELB)
    
    2. an *application cookie*, which is,
       
       - Generated by the load balancer,
       
       - Cookie name is <span style="color:orange">AWSALBAPP</span>.

- **Duration-based** cookies. (or load balancer generated cookie)
  
  - Cookie generated by the load balancer
  
  - Cookie name is <span style="color:orange">AWSALB</span> for ALB, <span style="color:orange">AWSELB</span> for CLB.

### Cross Zone Load Balancing

With <span style='color:limegreen'>**Cross Zone Load Balancing**</span>, each load balancer instance distributes evenly across all  registered instances in all AZ.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/elb-cross-zone-balancing.svg)

With <span style='color:blue'>**No Cross Zone Load Balancing**</span>, requests are distributed in the instances of the node of ELB.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/elb-no-cross-zone-balancing.svg)

#### Cross Zone Load Balancing Feature Availability on Load Balancer

- Application Load Balancer

  - Always on (can't be disabled)
  - No charges for inter AZ data

- Network Load Balancer

  - Disabled by default
  - You pay charges ($) for inter AZ data if enabled

  > You can access these settings by going to the `Attributes` section of the Load Balancer details.

- Classic Load Balancer

  - Disabled by default
  - No charges for inter AZ data if enabled

  > You can access these settings by going to the `Attributes` section of the Load Balancer details.

### SSL Certificates

-  An SSL Certificate allows traffic between your clients and your load balancer to be encrypted in transit (in-flight encryption).
- **SSL** refers to Secure Sockets Layer, used to encrypt connections,
- **TLS** refers to Transport Layer Security, which is a newer version.

> *TLS certificates are mainly used nowadays, yet people still refer as SSL.*

- Public SSL certificates are issued by Certificate Authority (CA). E.g., Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Letsencrypt, etc.
- SSL certificates have an expiration date (you set) and must be renewed.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/SSL.svg)

- The load balancer uses an X.509 certificate (SSL/TLS server certificate).
- We can manage certificate using ACM (AWS Certificate Manager).
- We can create upload our own certificates alternatively.
- HTTPS listeners:
  - A default certificate must be specified.
  - An optional list of certificates to support multiple domains.
  - Clients can use SNI (Server Name Indication) to specify the hostname they reach.
  - Ability to specify a security policy to support older versions of SSL/TLS (legacy clients)

#### SSL - Server Name Indication

- SNL solves the problem of loading multiple SSL certificates onto one web server (to server multiple websites).
- It's a *newer* protocol, and requires the client to **indicate** the hostname of the target server in the initial SSL handshake.
- The server will then find the correct certificate, or return the default one.

> Note:
>
> 1. Only works for ALB & NLB (newer generation, CloudFront)
> 2. Does not work for CLB (older generation)

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/sni-design.svg)

#### ELB - SSL Certificates

- **Classic Load Balancer (v1)**
  - Support only one SSL certificate.
  - Must use multiple CLB for multiple hostname with multiple SSL certificates.
  - **Hands-On**
    1. Click on your CLB and go to `Listeners`.
    2. Add another listener entry here with the following configuration:
       - *Load Balancer Protocol*: HTTPS
       - *Load Balancer Port*: 443
       - *Instance Protocol*: HTTP
       - *Instance Port:* 80
       - Select a *Cipher* (security cipher) for this entry.
       - Select a *Certificate* (support only one certificate)
       - Save the listener.
- **Application Load Balancer (v2)**
  - Supports mutliple listeners with multiple SSL certificates.
  - Uses SNI to make it work.
  - **Hands-On**
    1. Click on your load balancer and go to `Listeners`.
    2. Add another listener entry here with the following configuration:
       - *Protocol:Port*: HTTPS:443
    3. Add a *Default Action* (e.g., forward to target group, etc)
    4. Select a *Security Policy*.
    5. Select a *Default SSL Certificate*  (support only one certificate)
    6. We can repeat the steps 3 to 5 to add multiple certificates and corresponding default Actions.
    7. Save the listener.
- **Network Load Balancer (v2)**
  - Supports mutliple listeners with multiple SSL certificates.
  - **Hands-On**
    1. Click on your load balancer and go to `Listeners`.
    2. Add another listener entry here with the following configuration:
       - *Protocol:Port*: TLS:443
    3. Add a *Default Action* (e.g., forward to target group, etc)
    4. Select a *Security Policy*.
    5. Select a *Default SSL Certificate*  (support only one certificate)
    6. We can repeat the steps 3 to 5 to add multiple certificates and corresponding default Actions.
    7. Save the listener.

### Connection Draining

>  Feature naming :
>
> - Connection Draining - for CLB
> - Deregistration Delay - for ALB & NLB

- Time to complete "in-flight requests" while the instance is de-registering or unhealthly.
- Stops sending new requests to the EC2 instance which is de-registering.
- Between 1 to 3600 seconds (default: 300 seconds)
- Can be disabled (set value to 0)
- Set to a low value if your requests are short

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/connection-draining.svg)

## CLB (v1)

- Supports TCP (layer 4), HTTP & HTTPS (layer 7)

- Health checks are TCP or HTTP based

- Fixed hostname: `XXX.region.elb.amazonAWS.com`

**Hands-On**

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/clb-hands-on.svg)

1. Launch an EC2 instance with a application running.

2. Go to `Load Balancers`. 

3. Create a load balancer > Click on `Classic Load Balancer`.

4. Enter details in `Define Load Balancer`.
   
   1. Within `Basic Configuration`, 
      
      - Select `Create an internal load balancer` if you want to make a private lb.
        
        > Next we have protocol:port rules.
        > 
        > | Key                    | Value   |
        > | ---------------------- | ------- |
        > | LB Protocol:Port       | HTTP:80 |
        > | Instance Protocol:Port | HTTP:80 |
   
   2. Assign a Security Group.
   
   3. Configure Security Settings.
   
   4. Configure Health Check. (*health check response timeout < health check interval*)
   
   5. Add EC2 Instances.
   
   6. Add Tags.
   
   7. Click to create.

> The instances moves from <span style="color:orange">OutOfService</span> -> <span style="color:orange">InService</span>.

Open the DNS name from the CLB info section.

> *What could be wrong if say my instances are still* <span style="color:orange">OutOfService</span> ?
> 
> Possibly check security groups if say the HTTP rules are not present in the Inbound rules, the LB would not be able to forward requests to the instances.
> 
> *Ok, but right now I can access my instance directly from outside of the internet as well from the LB, how should I tighten the security?*
> 
> You can change security groups Inbound rules, by delete the existing HTTP rule <span style="color:orange">Source</span>, and then create a new rule with the <span style="color:orange">Source</span> being the CLB security group.

## Application Load Balancer (v2)

- ALB is Layer 7 (HTTP).

- It supports:
  
  - Load balancing to multiple HTTP apps across machines (target groups)
  
  - Load balancing to multiple apps on the same machine (ex. containers)

- It supports:
  
  - For HTTP/2 and Websocket
  
  - Redirects (from HTTP to HTTPS)
  
  - Routing table to different target groups:
    
    - Routing based on path in URL (example.com/users & example.com/posts)
    
    - Routing based on hostname in URL (one.example.com & other.example.com)
    
    - Routing based on Query strings, Headers (example.com/users?id=123&order=false)
  
  - ALB is a great fit for microservices & container-based application (example, Docker, Amazon ECS)
  
  - Supports a port mapping feature to redirect to a dynamic port in ECS.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/alb-design.svg)

> <span style="color:cyan">Good to know:</span>
> 
> - On creating an ALB, we get a fixed hostname (XXX.region.elb.amazonaws.com).
> - The application servers don't see the IP of the client directly. In order for the EC2 instance to look for client IP, we need to look for the following:
>   - The true IP of the client is inserted in the header **X-Forwarded-For**
>   - We can also get Port (**X-Forwarded-For**) and proto (**X-Forwarded-Proto**)
> 
> ![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/alb-request-forwarding.svg)

### Target Group Possiblities for ALB

- can be:
  
  - EC2 instances (can be managed by an Auto Scaling Group) - HTTP
  
  - ECS tasks (managed by ECS itself) - HTTP
  
  - Lambda functions - HTTP request is translated into a JSON event.

- Target groups must have IP addresses - must be private IPs.

- ALBs can route to multiple target groups.

- Health checks are at the target group level.

### Query strings/parameters routing

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/alb-query-based-routing.svg)

**Hands-On**

1. Launch an EC2 instance with a application running.

2. Go to `Load Balancers`.

3. Create a load balancer > Click on `Application Load Balancer`.

4. Select a `Scheme`.
   
   - A `Scheme` can be `Internet-facing` (routes requests from clients over the internet to targets. Requires a public subnet).
   
   - A `Scheme` can be `Internal` (routes requests from clients to targets using private IP addresses).

5. Select an IP address type.
   
   - IPv4
   
   - Dualstack (IPv4 and IPv6)

6. Select under Network mapping.
   
   1. Select VPC.
   
   2. Select Mappings (we need to select atleast 2 AZs and one subnet per zone, subnets can't be changed after the lb is created.)

7. Select a security group.

8. Specify Listeners and Routing.
   
   - Provide a target group.

9- Create the ALB.

## Network Load Balancer (v2)

- Network load balancers (Layer 4) allow to :
  
  - Forward TCP and UDP traffic to your instances
  
  - Handle millions of requests per seconds
  
  - Less latency ~100ms (vs 400 ms for ALB)

- NLB has one static IP per AZ, supports assigning Elastic IP (helpful in whitelisting specific IP)

- NLB are used for extreme performance, TCP or UDP traffic.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/nlb-design.svg)

### Target Group Possiblities for NLB

- EC2 Instances

- IP Addresses - must be private IPs

- ALB

**Hands-On**

- Launch an EC2 instance with a application running.

- Go to `Load Balancers`.

- Create a load balancer > Click on `Network Load Balancer`.

- Select a `Scheme`.
  
  - A `Scheme` can be `Internet-facing` (routes requests from clients over the internet to targets. Requires a public subnet).
  
  - A `Scheme` can be `Internal` (routes requests from clients to targets using private IP addresses).

- Select an IP address type.
  
  - IPv4
  
  - Dualstack (IPv4 and IPv6)

- Select under Network mapping.
  
  1. Select VPC.
  
  2. Select Mappings (we need to select atleast 2 AZs and one subnet per zone, subnets can't be changed after the lb is created.)
  
  >  For security group, the sg for instances is itself undertaken by the NLB.

- Specify Listeners and Routing.
  
  - Provide a target group.

- Create the NLB.

> If we directly try to open the URL of NLB, it would be inaccessible, the probable reason would be that the instances are unhealthly. In order to fix that, edit the security groups of the target group, to add in an <span style="color:orange">Inbound Rule</span>, for <span style="color:orange">HTTP</span> type traffic  with <span style="color:orange">TCP</span> Protocol under port <span style="color:orange">80</span>, allowing Source as 0.0.0.0/0 (anywhere), which is required for NLB.

## Gateway Load Balancer

- It is used to deploy, scale and manage a fleet of 3<sup>rd</sup> party network virtual appliances in AWS.

- It operates at Layer 3 (Network Layer) - IP packets.

- It combines the following functions:
  
  - **Transparent Network Gateway** - single entry/exit for all traffic.
  
  - **Load Balancer** - distributes traffic to your virtual appliances.

- It uses the **GENEVE** protocol on port 6081.

- Example: Firewalls, Intrusion Detection and Prevention Systems, Deep Packet Inspection Systems, payload manipulation, etc.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/05-aws-fundamentals-elb-and-asg/assets/gwlb-design.svg)

### Target Group Possiblities for NLB

- EC2 Instances

- IP Addresses - must be private IPs

- ALB

### Auto Scaling Group Overview

The goal of an ASG is to:

1. Scale out (add EC2 instances) to match an increased load.
2. Scale in (remove EC2 instances) to match a decreased load.
3. Ensure we have a minimum and a maximum number of machines running.
4. Automatically register new machines to a load balancer.

```
# minimum <= # desired <= # maximum
```

#### Attributes

An ASG has the following attributes:

1. A launch configuration
   - AMI + Instance Type
   - EC2 User Data
   - EBS Volumes
   - Security Groups
   - SSH Key Pair
2. Min size / Max size / Initial capacity
3. Network + subnets information
4. Load balancer information
5. Scaling policies

#### Auto scaling alarms

- It is possible to scale  an ASG based on CloudWatch alarms.
- An Alarm monitors a metrics (such as average CPU)
- <u>Metrics are computed for the overall ASG instances.</u>
- Based on the alarm:
  - We can create scale-out policies (increase instance #)
  - We can create scale-in policies (decrease instance #)

#### Auto Scaling New Rules

- It is now possible to define "better" auto scaling rules that are directly managed by EC2.
  - Target Average CPU Usage
  - Requests # per instance on the ELB
  - Average Network In
  - Average Network Out
- These rules are easier to set up and can make more sense.

#### Auto Scaling Custom Metric

- We can auto scale based on a custom metrics (ex: # connected users)
- In order to do that:
  1. Send custom metric from application on EC2 to CloudWatch (PutMetric API)
  2. Create CloudWatch alarm to react to low/high values.
  3. Use the CloudWatch alarm as the scaling policy for ASG.

> Note:
>
> 1. *Scaling policies can be on CU, Network, ... and can even be on custom metrics or based on a schedule (if you know your visitors patterns).*
> 2. ASGs use Launch configuration or Launch Templates (newer).
> 3. To update an ASG,  you must provide a new launch configuration / launch template.
> 4. IAM roles attached to an ASG will get assigned to EC2 instances.
> 5. ASGs are free. You pay for the underlying resources being launched,
> 6. Having instances under an ASG means that if they get terminated for whatever reason the ASG will automatically **create new onces as a replacement**. Extra safety !
> 7. ASG can terminate instances marked as unhealthy by an LB (and hence replace them).

### Scaling Policies

Types:

1. Target Tracking Scaling
   - E.g., I want the average ASG CPU to stay at around 40 %
2. Simple/Step Scaling
   - When a CloudWatch alarm is triggered (e.g., CPU > 70%), then add 2 units.
   - When a CloudWatch alarm is triggered (e.g., CPU < 30%), then remove 1 unit.
3. Scheduled Actions
   - Anticipate a scaling based on known usage patterns.
   - E.g., increase the min capacity to 10 at 5 pm on Fridays.
4. Predictive Scaling
   - Continuously forecast load and schedule scaling ahead

#### Good metrics to scale on 

- **CPU Utilization**: Average CPU utilization across your instances.
- **RequestCountPerTarget**: To make sure the number of requests per EC2 instances is stable.
- **AverageNetworkIn**/**AverageNetworkOut** (if you're application is network bound)
- **Any custom metric** (that you push using CloudWatch)

#### Scaling Cooldowns

- After a scaling activity happens, you are in the **cooldown period (default 300 seconds)**.
- During the cooldown period, the ASG will not launch or terminate additional instances (to allow for metrics to stabilize)

> Note:
>
> *Use a ready-to-use AMI to reduce configuration time in order to be serving request fasters and reduce the cooldown period.*
>
> Scaling down action is triggered usually within 15 minutes.

## Questions (important topics):

1. ***ASG Default Termination Policy***:

   - Find the AZ which has the most number of instances.
   - If there are multiple instances in the AZ to choose from, delete the one with the oldest launch configuration.

   > ASG tries to the balance the number of instances across AZ by default.

2. ***Lifecycle hooks***:

   - By default as soon as an instance is launched in an ASG it's in service.

     ![](https://docs.aws.amazon.com/autoscaling/ec2/userguide/images/lifecycle_hooks.png)

   - You have the ability to perform extra steps before the instance goes in service (pending state).
   - You have the ability to perform some actions before the instance is terminated (terminating state).

3. ***Launch template vs launch configuration***

   - Both allow to change:
     - ID of AMI, 
     - Instance type, 
     - Key pair,
     - Security Groups,
     - Tags,
     - EC2 user data, etc.

   | Launch Configuration           | Launch Template                                              |
   | ------------------------------ | ------------------------------------------------------------ |
   | Must be re-created every time. | Can have multiple versions.                                  |
   |                                | Create parameters subsets (partial configuration for re-use and inheritance) |
   |                                | Provision using both On-Demand and Spot instances (or a mix) |
   |                                | Can use T2 unlimited burst feature.                          |
   |                                | *Recommended by AWS going forward.*                          |

   4. For compliance purposes, you would like to expose a fixed static IP address to your end-users so that they can write firewall rules that will be stable and approved by regulators. What type of Elastic Load Balancer would you choose? 
      ***Network Load Balancer***

   5. You have an Auto Scaling Group fronted by an Application Load Balancer. You have configured the ASG to use ALB Health Checks, then one EC2 instance has just been reported unhealthy. What will happen to the EC2 instance?
      ***The ASG will terminate the EC2 instance***

   6. A web application hosted on a fleet of EC2 instances managed by an Auto Scaling Group. You are exposing this application through an Application Load Balancer. Both the EC2 instances and the ALB are deployed on a VPC with the following CIDR `192.168.0.0/18`. How do you configure the EC2 instances' security group to ensure only the ALB can access them on port `80`?

      ***Add an Inbound Rule with port `80` and ALB's Security Group as the source***

      



