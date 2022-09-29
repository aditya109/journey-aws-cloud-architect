# Route 53

Table of Contents



## What is DNS ?

DNS or Domain Name System which translates the human friendly hostnames into the machine IP addresses.

www.google.com => 172.217.18.36

DNS uses hierarchical naming structure.

- .com
- example.com
- www.example.com
- api.example.com

**Terminologies**

1. **Domain Registrar**: Amazon Route 53, GoDaddy, etc.
2. **DNS Records**: A, AAAA, CNAME, NS, etc.
3. **Zone File**: contains DNS records.
4. **Name Server**: resolves DNS queries (Authoritative or Non-Authoritative)
5. **Top Level Domain (TLD)**: .com, .us, .in, .gov, .org, etc.
6. **Second Level Domain (SLD)**: amazon.com, google.com, etc.

http://api.www.example.com.

1. The last dot after `com` is the <span style="color:brown">**root**</span> domain.
2. The second last dot before `com` and after `example` is <span style="color:skyblue">**top-level**</span> domain.
3. The third last dot before `example` and after `www` is <span style="color:lightgreen">**second-level**</span> domain.
4. The fourth last dot before `www` and after `api` is <span style="color:purple">**sub-level**</span> domain.
5. The entire thing when combined without the protocol is <span style="color:crimson">**domain** </span>name, i.e., <span style="color:skyblue">`api.www.example.com`</span>.
6. Lastly, `http` is your protocol.
7. Combining domain name and protocol, we get FQDN or *Fully Qualified Domain Name*.

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/07-route-53/assets/dns.svg)

## Overview- Route 53

Amazon Route 53 is a highly available, scalable, fully managed and Authoritative DNS. *Authoritative = the customer (you) can update the DNS records.*

Route 53 is also a Domain Registrar.

- It also has ability to check the health of your resources.
- The only AWS service which provides 100% availability SLA.
- Why the name *Route 53* ? 53 is a reference to the traditional DNS port.

### Route 53 - Records

- They can details of how you want to route traffic for a domain.

- Each record contains:
  1. **Domain/subdomain name** - example.com
  2. **Record Type** - A or AAAA
  3. **Value** - 12.34.56.78
  4. **Routing Policy** - how Route 53 responds to queries
  5. **TTL** - amount of time the record cached at DNS resolvers

- Route 53 supports the following DNS record types:
  - *must know* - A / AAAA / CNAME / NS
  - *advanced* - CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV

#### Record Types:

| Record Type | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| A           | maps a hostname to IPv4                                      |
| AAAA        | maps a hostname to IPv6                                      |
| CNAME       | maps a hostname to another hostname.  <br />- *The target is a domain name which must have an A or AAAA record.*<br />- *Can't create a CNAME record for the top node of a DNS namespace (Zone Apex)*<br />*Example, you can't create a domain for example.com, but you can create for www.example.c*om |
| NS          | Nameservers for the Hosted Zone<br />- *Control how traffic is routed to a domain.* |

#### Hosted Zones

- A container for records that define how to route traffic to a domain and its sub-domain.
  The types of hosted zones are:
  - **Public Hosted Zones** - contains records that specify how to route traffic on the Internet (public domain names). They can answer queries from public clients.
    Example, <span style="color:blue">application1.mypublicdomain.com</span>
  - **Private Hosted Zones** - contains records that specify how you route traffic within one or more VPCs (private domain names). They allow you to identify private resources with private names within private VPC.
    Example, <span style="color:blue">application1.company.internal</span>
- You pay $0.5 per month per hosted zone.

### Registering a Domain

### Creating first records

### EC2 setup

### Records TTL (Time to Live)

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/07-route-53/assets/records-ttl.svg)

- If we have high TTL, e.g., 24 hours
  - Less traffic on Route 53
  - Possibly outdated records
- If we have low TTL, e.g., 60 secs
  - More traffic on Route 53 (expensive)
  - Records are outdated for less time
  - Easy to change records.

> <span style="color:orangered">Except for Alias records, TTL is mandatory for each DNS record.</span>

### CNAME vs Alias

- AWS resources (Load Balancer, CloudFront, etc) expose an AWS hostname.

- CNAME:

  - Points a hostname to any other hostname. (app.mydomain.com => blabla.anything.com)

  > <span style="color:orangered">**ONLY FOR NON-ROOT DOMAIN (aka something.mydomain.com)**</span>

- Alias:

  - Points a hostname to an AWS Resource. (app.mydomain.com => blabla.amazon.com)
  - Free of charge
  - Native health check capability within itself.
  - Is an extension to DNS functionality, automatically recognizes changes in the resource's IP addresses.
  - Unlike CNAME, it can be used for the top node of a DNS namespace (Zone Apex) e.g., example.com
  - Alias Record is always of type A/AAAA for AWS resources (IPv4/IPv6) 
  - Effective targets for Alias records
    - ELBs
    - CloudFront Distributions
    - API Gateway
    - Elastic Beanstalk environments
    - S3 Websites
    - VPC Interface Endpoints
    - Global Accelerator 
    - Route 53 record in the same hosted zone

  > - <span style="color:orangered">**Cannot set an Alias records from an EC2 DNS name**</span>
  > - <span style="color:orangered">**Cannot set TTL**</span>
  > - <span style="color:orangered">**Works for both ROOT and NON-ROOT DOMAIN (aka  mydomain.com)**</span>

### Routing Policies

This defines how Route 53 responds to DNS queries.

> NOT SAME AS `ROUTING`.
>
> It is not same as LB routing which routes the traffic. 
> DNS does not route any traffic it only responds to the DNS queries.

Route 53 supports the following routing policies:

1. Simple
2. Weighted
3. Failover
4. Latency based
5. Geolocation
6. Multi-value answer
7. Geoproximity 

#### Simple

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/07-route-53/assets/simple-routing-policy.svg)

- Typically routes traffic to a single resource.
- Can specify multiple values in the same record

> <span style="color:crimson">If multiple values are returned, a random one is chosen by the client.</span>

- When Alias is enabled, specify only one AWS resource.
- Can't be associated with Health checks.

#### Weighted

- Control the percentage of the requests that go to each specific resource.

- Assign each record a relative weight:

  - traffic (%) = Weight from a specific record / Sum of all the weights for all records

  > *Weights don't need to sum up to 100, rather 256.* Each can be between 0 and 255 

- DNS records must have the same name and type.

- Can be associated with Health Checks.

- Use cases: load balancing between regions, testing new application versions...

- Assign a weight of 0 to a record to stop sending traffic to a resource.

- If all records have weight of 0, then all records will be returned equally.

#### Latency

- Redirect to the resource that has the least latency close to us.
- Super helpful when latency for the users is a priority.
- Latency is based on traffic between users and AWS Regions.
  For example, Germany users may be directed to the US.
- Can be associated with Health Checks (has a failover capability)

#### Health Checks

- HTTP Health Checks are only for **public resources**.

- Health Check => Automated DNS Failover.

  - Health checks that monitor an endpoint (application, server or other AWS resources)

    - About 15 global health checkers will check the endpoint health.

      - Healthy/unhealthy threshold - 3 (default)
      - Interval - 30 sec (can be set to 10 sec - higher cost)
      - Supported protocol - HTTP, HTTPS and TCP
      - If count of health checkers greater than 18% report the endpoint is healthy, Route 53 considers it healthy, else unhealthy.
      - Ability to choose which location you want Route 53 to use.

      > <span style='color:orangered'>Required configuration on router/firewall to allow incoming from Route 53 Health Checks (within application instance).</span>

    - Health checks pass only when the endpoint responds with the `2xx` and `3xx` status codes.

    - Health checks can be setup to pass/fail based on the text in the first **5120 bytes** of the response.

  - Health checks that monitor other health check (calculated health checks)

    - Combine the results of multiple health checks into a single health check using using OR, AND or ATLEAST.
    - Can monitor upto 256 child health checks.
    - Specify how many of the health checks need to pass to make the parent pass.
    - Usage: Perform maintenance to your website without causing all the health checks to fail.

- Health checks that monitor CloudWatch Alarms (full control !!) - e.g., throttles of DynamoDB, alarms on RDS, custom metrics (helpful for private resources.)

  - Route 53 health checkers are outside the VPC, meaning they can't access **private** endpoints (private VPC or on-premise resource)

  - We can create a **CloudWatch Metric** and associate a **CloudWatch Alarm**, then create a Health Check that checks the alarm itself.

- Health checks are integrated with CW metrics.

#### Failover (Active-Passive)

![](https://github.com/aditya109/journey-aws-cloud-architect/raw/main/07-route-53/assets/failover-routing-policy.svg)

#### Geolocation

- Different from latency-based. This is based on user location.
- Specify location by continent, country or by US State (if there's overlapping most precise location is selected.)
- Should create a `Default` record (in case of no-match)
- Can be associated with health checks
- Usecase: website localization, restrict, content distribution, load balancing, etc.

#### Traffic Flow and Geoproximity

- Route traffic to your resources based on the geographic location of users and resources.

- Ability to ***shift more traffic to resources*** based on the defined ***bias***.

- To change the size of the geographic region, specify bias values:

  - To expand (1 to 99) - more traffic to the resource
  - To shrink (-1 to -99) - less traffic to the resource

- Resources can be :

  - AWS resources (specify AWS region)

    > <span style='color:orangered'>Route 53 Traffic Flow is required for using this feature.</span> 

#### Multi-value

- Use when routing traffic to multiple resources

- Route 53 return multiple values/resources

- Can be associated with Health checks (return only values from healthy resources)

- Upto to 8 healthy records are returned from each Multi-Value query.

  > **<span style='color:orangered'>Multi-value is not a substitute from having an ELB.</span>**
  >
  > This is different from simple multiple value record, as the later does not allow health checks.

## 3rd Party Domains & Route 53

### Domain Registrar vs DNS Service

- You can buy or register your domain name with a Domain Registrar typically by paying annual charges. (e.g., GoDaddy, Amazon Registrar, etc.)
- The Domain Registrar usually provides you with a DNS service to manage your DNS records.
- But you can use another DNS service to manage your DNS records.
- Example: purchase the domain from GoDaddy and use Route 53 to manage your DNS records.
- You create the Nameserver on AWS and linked it to your domain.

> <span style='color:orangered'>Domain Registrar != DNS Service (former usually comes with some parts of latter).</span>

## Questions

| You have deployed a new Elastic Beanstalk environment and would like to direct 5% of your production traffic to this new environment. This allows you to monitor for CloudWatch metrics and ensuring that there are no bugs exist with your new environment. Which Route 53 Record type allows you to do so?<br />Options: SIMPLE, WEIGHTED, LATENCY, FAILOVER |
| ------------------------------------------------------------ |
| Ans. WEIGHTED                                                |
| **You have updated a Route 53 Record's myapp.mydomain.com value to point to a new Elastic Load Balancer, but it looks like users are still redirected to the old ELB. What is a possible cause for this behavior?** |
| Ans. Because of TTL                                          |
| **You have an application that's hosted in two different AWS Regions `us-west-1` and `eu-west-2`. You want your users to get the best possible user experience by minimizing the response time from application servers to your users. Which Route 53 Routing Policy should you choose?** |
| Ans. Latency                                                 |
| **You have a legal requirement that people in any country but France should NOT be able to access your website. Which Route 53 Routing Policy helps you in achieving this?** |
| Ans. Geolocation                                             |
| **You have purchased a domain on GoDaddy and would like to use Route 53 as the DNS Service Provider. What should you do to make this work?** |
| Ans. Create a Public Hosted Zone and update the 3rd party Registrar NS records. |

