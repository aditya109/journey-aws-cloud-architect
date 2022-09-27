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

### TTL

### CNAME vs Alias

### Routing Policies

#### Simple

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



