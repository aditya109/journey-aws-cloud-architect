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

## Overview

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



