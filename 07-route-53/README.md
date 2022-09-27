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

#### Latency

#### Health Checks

**Hands On**

#### Failover

#### Geolocation

#### Traffic Flow and Geoproximity

**Hands-On**

#### Multi-value

## 3rd Party Domains & Route 53



