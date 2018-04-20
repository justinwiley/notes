# Google Innovations in Networking

This contains my notes and analysis of Google's networking strategy and a review of "Google Cloud Networking Deep Dive" from 2017:

- https://www.youtube.com/watch?v=nRJBFNyY944

### Key Innovation: Abstraction Breaking and Software-Defined Everything

Google wanted and needed a way to dynamically group resources on-demand.  

Big-Query and Spanner have to be understood in the context of these efforts to build networks on the fly that are:

- Location independent
- Reliable, recoverable and resilient
- Divorced to the greatest extent possible from network fabric

## History of Innovations

- Google Global Cache
- [Maglev](https://cloudplatform.googleblog.com/2016/03/Google-shares-software-network-load-balancer-design-powering-GCP-networking.html) (2009) - Software Defined Load Balancer
- B4 (2011) - Software Defined WAN
- [Jupiter](https://cloudplatform.googleblog.com/2015/08/a-visual-look-at-googles-innovation-in.html) (2012) - made possible Software Defined Data Centers
- Andromeda (2014) - Software Defined Network Virtualization
- Espresso (2017?) - Software Defined Edge

There seems to be a logical and physical dimension to both of these.  **Physically**, this involved pushing the boundaries of networking fabric, in projects like Firehose, Watchtower and Jupiter.

```
Unlike the custom Jupiter fabrics that carry traffic around Google’s data centers, Maglev load balancers run on ordinary servers — the same hardware that the services themselves use.
```

- ["Google shares software network load balancer design powering GCP networking"](https://cloudplatform.googleblog.com/2016/03/Google-shares-software-network-load-balancer-design-powering-GCP-networking.html)

**Logically**, there's the idea of doing as many traditional hardware things in software on commodity hardware as possible (networks, firewalls, load-balancing, DNS).  

[Borg / Kubernetes](https://kubernetes.io/blog/2015/04/borg-predecessor-to-kubernetes) was crucial to these developments, and was (IMHO) an effort towards breaking the reliance

## Organizations and Projects

Google's ["Organization Nodes"](https://cloud.google.com/resource-manager/docs/quickstart-organizations) and [projects](https://cloud.google.com/resource-manager/docs/creating-managing-projects) is the primary unit for grouping services in GCP.

- Organization Node (myorg.com)
- [Shared VPC (XPN)](https://cloud.google.com/vpc/docs/shared-vpc)
    - Subnet 10.0.0.0/20
        + Project Dev Team
            * Instances
        + Project Dev Team 2
            * Instances

https://www.coursera.org/learn/gcp-infrastructure-foundation/lecture/vUIy6/cloud-virtual-network-cvn-objects

### Projects

Projects are a "virtual container".

- They contain entire networks
- Up to 5 public-facing total

### Networks

Networks don't have an IP address range.  They are a *concept*, logically separating resources.  Networks are global, they span all regions across the globe.

Isolating at the switch level or the route level goes away.

us-east1, asia-east2 can exist in a single network

- Any services in a network looks like they are one hop away
    + This applies to internal and external IPs
- When one Google network sends data to another, it still has to *egress*
    + Egressing connects through edge routers
    + Has different security and billing ramifications
    + *Egress* data charges will apply

#### Types of Neworks

- Default
- Auto
    + IP address ranges assigned
    + Default firewall rules
- Custom
    + IP address customizable
    + No default firewall rules

### Subnetworks

Subnets exist in a single region, and are used to cross zones in those regions.

- A single firewall rule applies to that subnet
- Subnets are used to manage resources, since they like larger networks have no IP ranges
- They can represent departments, businesses, or functions

| Marketing Subnet | ERP Subnet | Development Subnet |
| ------------- | ------------- | ------------- |
| 10.1.1.0 | 192.133.71.0 | 192.167.45.13 |
| (VM) | (VM) | (VM) |
| (VM) | (VM) | - |
| - | (VM) | - |
| - | (VM) | - |

The physical network doesn't have to match this logical network.

### IP addresses

Each VM assigned two IP addresses *Internal* and *External*.

| Internal IP | External IP |
| ------------- | ------------- |
| Allocated from subnet range to VMs by DHCP | Assigned from Pool (ephemeral) |
| DHCP reservation is renewed every 24 hours | Reserved (static), Billed when not attached to a running VM |
| VM Name + IP is registered with network-scoped DNS | VM doesn't know external IP, it's mapped to internal IP |

### DNS Resolution

#### Internal

Each instance has a fully-qualified domain name that points to it's internal IP:

```
<hostname>.c.<project-id>.internal
```

- Handled by internal Google DNS

#### External

Google cloud has managed DNS.

### Routes

Routes map traffic to destination networks.

- Applies to traffic egressing a VM
- Forwards traffic to most specific route
- Traffic only delivered if it also matches a firewall rule
- Created when subnet is created
- Applies to tagged VMs or all VMs
- Enables VMs on same subnet to communicate

### Firewall Rules

- Positive / white list based, only ALLOW, no DENY statements
- IP based, but *labels* are preferred.  Easy way to specify firewall rules for lots of VMs uniformly regardless of IP
- Egress rules to prevent exfiltration of data as well
- Default rules for auto-type networks, not custom
- Prioritizable rules

### Routes and Firewall Rules

Both are network resources

- They cant be shared between projects
- They are maintained in network tables
- No physical routing, software defined "traffic engineering"

### Billing

- You are billed for traffic *egress*
    + To the Internet
    + Or from one region to another (lower fee)
    + Between zones within a region
- Approximate
    + 1 cent per gigabyte between regions
    + 8 cent per gigabyte for egress to Internet
- Not billed for:
    + Traffic *ingress*
    + VM to VM in the same zone (same region, same network)
    + traffic to GCP services, limits apply

### Throughput and Latency

- Varies with location
- Co-locate within a region for performance
- *Any feature with Beta has no SLA*

## Common Network Patterns

#### Availability - Two VMs in two different zones with one subnet

- Redundancy
- Same firewall

#### Security - Two Subnets

- Isolates VMs, creates a security boundary between them

#### Security + Availability - Multiple zones, multiple subnets

- Isolates traffic, provides redundancy
- Think about latency and data costs: dont put app servers in one zone and db servers in another

#### Globalization - Multiple Regions

- Create networks within multiple regions
- Use load balancers to send traffic to users closest to given region
- Allows you to meet regulatory constraints

#### Isolation - Two Distinct Networks

- If two networks need to be totally separated, use two separate networks
- Each network accessible from the internet, no cross-network traffic at any level

#### Management Isolation - Multiple Projects, Multiple Networks

- Multiple teams, multiple projects
- Separate IAM, networks, routing etc
- Separate billing
- Can still manage under an "organization scope"

#### Bastion Host Isolation

- Instance used as jump host with logs and auditing
- Proxies SSH traffic to internal network
- Firewall rules restrict SSH to bastion

#### NAT Gateway Isolation

- Two or more networks, only accessible through external IP address
- Networks


