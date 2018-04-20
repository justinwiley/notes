# Google Cloud Platform (GCP) Fundamentals

This is a review of the Coursera course:

- https://www.coursera.org/learn/gcp-fundamentals

## Overview

### My General Thoughts on Google and GCP

Google went through multiple internal phases of technology before productizing their tool-chain in the form of GCP.  Applications drove these technology phases, Google Search and Ads primary among them.  All of these capabilities were also driven by Google's mission to ["organize the world's information and make it universally accessible and useful."](https://www.google.com/search?q=googles+mission&oq=googles+mission&aqs=chrome..69i57j0l5.1737j0j7&sourceid=chrome&ie=UTF-8).  They were also driven by a strategic commitment to [extreme flexibility](https://hbr.org/2015/08/google-couldnt-survive-with-one-strategy), to pursue a huge number of goals at scale, they needed infrastructure that could bend in many directions without breaking.

Each new feature drew on previous understandings, product and marketing goals, and Google Research's projection of future technical possibilities.

The resource hierarchy and identities and roles play a more obvious role than in AWS.  Organizations create folders that have projects is something that seems to take place first and be more methodical than the usual AWS pattern of "spin up a bunch of services and lock them down later."

### Initial Beginnings

Google realized early that making this technology available externally made sense for a number of reasons, including the classic open-source strategy of building [a shared ecosystem of value](https://hbr.org/2016/10/the-ecosystem-of-shared-value) around it's technology.  It started to open-source and productize pieces of this infrastructure, starting with [AppEngine in 2008](https://en.wikipedia.org/wiki/Google_Cloud_Platform).  This was 2 years after [Amazon's Elastic Compute Cloud](https://techcrunch.com/2016/07/02/andy-jassys-brief-history-of-the-genesis-of-aws/), and the success of that platform was [undoubtedly a big influence](https://plus.google.com/+RipRowan/posts/eVeouesvaVX).

GCP is also designed to encompass GSuite, in ways that are simultaneously useful, and somewhat unwieldy.

### Difference from other Cloud Services

"Every company is a data company."  Google Cloud fundamentally sees the future of applications, and meaningful work in general, as a data processing problem.  The implication perhaps is that other cloud services like Amazon are focused on running traditional existing tasks and infrastructures, like multi-tier application servers backed by OLTP and OLAP, and not prepared for a data-first future.

### Spectrum of Services

Google Cloud puts it's service offerings on the spectrum of managed infrastructure, offering a variety of ways to meet existing customer workloads.

| Service | Level of Management |
| ------------- | ------------- |
| Compute Engine | IaaS |
| Kubernetes Engine | Hybrid |
| App Engine | PaaS |
| Cloud Function | Serverless Logic |
| Managed Servies | Fully-managed elastic Resources |

### 4 Primary Services:

According to the presenter, there are 4 core services

- Compute
    + Compute Engine
    + Kubernetes
    + App Engine
    + Cloud Functions
- Storage
    + BigTable
    + Storage
    + Firebase
    + CloudSQL
    + Spanner
    + DataStore
- Big Data
    + BigQuery
    + Pub/Sub
    + Dataflow
    + Dataproc
    + Datalab
- Machine Learning
    + NLP API
    + Vision API
    + Machine Learning
    + Speech API
    + Translate API

This contradicts somewhat with other sources like the [Google Cloud Networking Deep Dive](https://www.youtube.com/watch?v=nRJBFNyY944), which lifts networking to that tier, an idea which I agree with.

### Locality

Google has internally been globally oriented from it's inception, and as such it's services and infrastructure life in explicit geographic domains.

The hierarchy is:

```
Multi-Region -< Region -< Zones
```

#### Multi-Region

- "Europe"
- At least two regions, separated by "at least 160 kilometers"

#### Region

- "europe-west-2"
- 15 regions globally

#### Zone

- "europe-west-2a"

Which geographic domain(s) services live in depends on what their intention is.  For example block-level storage needs to be broadly available and resilient, and so it is necessarily multi-region.  User regions

### Pricing

Google charges by the second, and doesn't round up, which they believe is a pricing innovation that "adds-up."

- Billing in sub-hour increments - VMs, containers, data processing
- Discounts for sustained use - VM machines over 25% a month
- Custom VM instance types - on-demand pricing

### Open APIs

Google Cloud offers open APIs, generally to services developed internally in Google and released as open-source projects.  The idea is customers could theoretically deploy on Google cloud, and then leave and go somewhere else in the future, if they so choose.

### Security

Security is baked in.

- A variety of OpSec, red-team exercises, intrusion detection
- Internet communication - Google Front End, Built in DDoS protection
- Encryption at Rest and "automatic in-transit encryption" on the network
- User Identity, intelligent challenges based on a risk-profile
- Hardware protection, like ["Titan"](https://cloudplatform.googleblog.com/2017/08/Titan-in-depth-security-in-plaintext.html)

## Core Infrastructure

- *Resource hierarchy* - how Google logically groups and separates tasks, resources, users
- *Identity management* - who can connect, and how
- *Interacting with Google Cloud Platform* - the management layer used to interact with GCP

### Resource Hierarchy

https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy

- How you group your resources based on a structure
- Provides a hierarchy of ownership
- Provides "attach points" for inheritence
- "Policies are inherited downward in the hierarchy."
    + **But** policies applied at higher level cannot remove policies granted at lower levels.
    + So if a user is granted access to a storage bucket at a lower-level
- Levels of hierarchy provide trust boundaries and resource isolation

```
Org Node -< Folders -< Projects -< Resources
```

| Hierarchy Unit | Example |
| ------------- | ------------- |
| Organization | example.com |
| Project | Bookshelf, static-assets, stream-ingest |
| Resources | Storage, App Engine, SQL, Pub/Sub, etc |

#### Org Node

- Top of the hierarchy
- If you have a GSuite domain, GSuite projects will automatically belong to that organization node
- Or you can create it using IAM
- Roles:
    + Organization Policy Administrator - broad control over all resources
    + Project Creator - finer-grained control of project creation

#### Folders

- Different departments, teams, enviornments
- Delegate administrative types

#### Projects

- Projects - groups related resources with a common business objective

#### Resources

- Services like CloudSQL, Storage, App Engine, Cloud ML, Kubernetes etc

### IAM Policies

https://cloud.google.com/iam/

- "Lets administrators authorize who can take action on specific resources"
- *Who*
    - Google account, GSuite account, entire domain
    - Service account
        - Machine only, no login
        - For example, a Google Compute VM has the ability to manipulate other services
        - Uses cryptographic keys instead of passwords
    - Google group
    - Cloud Identity or G Suite Domain
- *Can do what* - Collection of operations, CRUD for example
- *On which resource* - Google account, google group, 

### Primitive Roles

- Owner, Editor, Viewer, Billing Administrator
- GSuite targeted coarse-grained levels of access
- ["roles that existed prior to the introduction of Cloud IAM"](https://cloud.google.com/iam/docs/understanding-roles#primitive_roles)

### Predefined Roles

- apply to a specific GCP service
- "InstanceAdmin" role, for example
    + compute.instances.delete
    + compute.instances.get
    + ...etc
- **only** work at the Organization and Project level, not at Folder level

### Using Service Accounts

- One pattern is to create a service accounts for each role a group of services needs
- I.e. you can grant different VMs in your project different identities
- For example, if the app server needs to adjust SQL resources, a service account would be created with that role

## Interacting with the Cloud Platform

### The management layer, aka *Google SDK*

"You use your choice of several interfaces to connect"...to GCP.

- The management layer strongly encourages a principle of least-principle is crucial
- In a least-privileged environment users are projected

It consists of:

- the Console - web interface
- Cloud Shell and SDK - CLI interface
- Console and Mobile App - iOS and Android
- REST-based API - for custom applications

#### Cloud Launcher

- Pre-packaged, ready-to-deploy services
- GCP updates base packages automatically

## Compute

### VPC

https://cloud.google.com/vpc/

[More here](google-cloud-networking.md)

- VPCs you define have **global scope**, they can have subnets in any region, and can span regions
- You have resources in resources in different zones on the same subnets
- This follows with Google Clouds general strategy of "software defined everything"
- Several different kinds of load-balancing
- Routing tables
    + Forward traffic within networks and even subnets!
    + Built-in
- Global distributed firewall
    + Incoming, outgoing
    + Can define with metadata rules
- Shared VPC - control entire network or subnet works
- VPC Peering - interconnect GCP projects

#### Cloud Load-Balancing

- Fully managed, global
- User get a single, global anycast IP address
- Traffic goes over the Google backbone from Edge nearest user
- Multi-region fail-over
- No pre-warming is required

| Type | Hierarchy |
| ------------- | ------------- |
| HTTP(S) Layer 7 | Global |
| HTTP(S) Layer 4 | Global |
| TCP | Global |
| UDP | Regional |
| Internal Load-balancing | Regional |

#### Cloud DNS

https://cloud.google.com/dns/

- Global managed DNS
- Redundant, programmable

#### CDN

https://cloud.google.com/cdn/

- Globally distributed Edge cache

#### Interconnect with other Networks

| Name | Opts |
| ------------- | ------------- |
| VPN | Secure communication across VPN tunnels |
| Direct Peering | Private connection between Google and your very own datacenter |
| Carrier Peering | Bigtime peering |
| Dedicated Interconnect | Super big-time peering, easily capable of oodle-flops per shmoodle plop |


### Compute (VMs)

https://cloud.google.com/compute/

- Runs on a pre-defined or custom machine type
- VMs can run windows, linux, customized versions
- Many GCP zones have GPUs available
- 2 kinds of persistent storage
    + Persistent: Standard or SSD
    + Temporary: Local SSD scratch-space
- Can define a start-up script
- Disk snapshots
- "Innovative pricing", preemptible VMs are cheaper if your task can be torn down and restarted at will
- Vertical and/or horizontal scaling
    + "Big VMs" with lots of CPU and ram
    + Auto-scaling for horizontal scale

### Cloud Storage

https://cloud.google.com/storage/

- Arbitrary bunch of bytes, not file storage, not mountable
    + Just a bunch of buckets
    + Although we can all pretend because it makes more sense
- Immutable!
    - Versioning can be turned on and off
    - If off, new always overwrites old
    - Can define a variety of rules about versioning and retention
- Encrypted at rest, server side.  Encrypted in-transit
- Full-managed
- Tied to a geographic location
- Direct Download - distributing large files to user
- Roles are inherited project -> bucket -> object
- Access Control Lists (ACLs) offer finer controls
- Buckets have global names

#### Classes

| Type | Multi-Regional | Regional | Nearline | Coldline |
| --- | --- | --- | --- | --- |
| Access | Most frequently | Accessed frequently within a region | Accessed less than once a month | Accessed less than once a year |
| Availability SLA | 99.95% | 99.90% | 99.00% | 99.00% |
| Use Case | Content storage and delivery | In-region analytics, transcoding | Long-tail content, backups | Achiving, disaster recovery |

- All types have millisecond access times, but cheaper storage types charge more to retrieve data
- Cost per gigabyte per month, multi-regional most expensive, coldline cheapest
- Coldline and nearline incur additional charge for read
- Online transfer, storage transfer service, transfer appliance

### BigTable

https://cloud.google.com/bigtable/

- NoSQL
- OLTP workloads
- sparsely populated
- fully-managed
- persistent hash-table
- low-latency, high throughput
- IOT, analytics, logs
- Same API as HBase
- Built on
    + Google File System
    + Chubby distributed lock
    + SSTable file format
- Same database that powers search, analytics and Gmail
- Streaming in (usually via Dataflow)
- Batch processing (via Hadoop, Dataflow, Spark streaming etc)

### CloudSQL

- MySQL and Postgres managed
- OLTP workloads
- Automatic replication, failover
- Multiple zones
- On-demand or scheduled backups
- Vertically by machine type and horizontally via read-replicas
- Accessible by external services

### Cloud Spanner

- SQL queries
- OLAP workloads
- horizontally scalable super-database
- If you need:
    - sharding for high performance
    - strong, transactional consistency
    - global data
* Usecases: financial applications and inventory applications

### Cloud Datastore

- another NoSQL storage
- SQLish
- typically used with AppEngine
- fully managed, automatic sharding, replication
- transactions for multiple rows, unlike BigTable

## Kubernetes and Containers

- VMs
    + Isolated below OS level via hypervisor
    + Has entire copy of an OS, slow to boot, CPU intensive
    + Stages
        * Provisioning
        * Staging
        * Running
            - Live migration to different zones!
        * Stopping
    + Zones support different architectures
- Container
    + Isolated at OS level via process isolation
        * Namespaces
        * Supervisor process can limit access of other processers
    + Faster to start, fewer resources
- Kubernetes uses Docker containers
- Why containers?
    + Consistency
    + Loose coupling
    + Workload Migration
    + Agility

### Kubernetes

[More on Kubernetes](from-borg-to-omega-to-kubernetes.md)

- Pods
    + Collections of containers
- Clusters
    + Group of machines (Nodes) where Kubernetes can schedule containers
- Nodes
    + Can be spread across zones for redundancy

#### Features

- Rolling updates
- Multi-zone clusters
- Load balancing
- Autoscaling
- Declarative


Kubernetes can automatically scale load as ingress changes

#### Kubernetes Engine

- Build, resize, delete clusters
- Resources come from Compute Engine
- Nodes are compute engine VMs
- Managed service
- Built in-logging
- Google updates and improves Kubernetes
- Nodes can be automatically uploaded

#### Google Cloud Container Builder

- Create Docker Container images from app code in Google Cloud Storage

#### Google Container Engine

- Docker image storage that's private to your GCP project

## App Engine

https://cloud.google.com/appengine/

- Fully managed, PaaS
- Automatic scaling

### Environments

- Standard
    + Simpler, fine-grained autoscale
    + Free daily quota, low utilization possibly at no charge
    + SDKs for testing locally and deploying
    + PHP
    + Java
    + Python
    + Go
    + Sandboxed, no other languages allowed
    + App cant write to filesystem, requests have 60 second timeout

- Flexible
    + Runs inside containers
    + No sandbox constraints
    + Can access app engine resources
    + Backward compatibility

## Cloud Endpoints and Apigee Edge

https://cloud.google.com/endpoints/

- Distributed API management through console
- Cloud access and validate calls with JWTs and Google API keys
- Identify users with Auth0 and Firebase Auth
- Generate client libraries

| Runtime Environment |
| ------------- |
| App Engine Flexible |
| Kubernetes |
| Compute Engine |

| Clients |
| ------------- |
| Android |
| iOS |
| Javascript |

### Apigee Edge

https://cloud.google.com/apigee-api-management/

- Monitize APIs
- Analytics, monitization, developer portal
- Can use Api edge to wrap legacy applications outside of GCP

## Cloud Functions **BETA** (shiver)

https://cloud.google.com/functions/

- Single purpose functions written in Javascript
- Billed in 100ms increments
- Trigger from:
    + HTTP
    + Pub/Sub
    + Cloud Storage
    + Firebase
    + Stackdriver logging

## Deployment Manager

- YAML Infrastructure as Code specification of environment
- You can store and version your templates in Google Cloud Source Repository!

## Monitoring / Stackdrive

https://cloud.google.com/stackdriver/

- Monitoring
- Logs
- Metrics
- Traces
- Error reporting
- Connects source code application state to logs

## Big Data

### Cloud DataProc

- Managed Hadoop, Spark, Pig
- Clusters in 90 seconds
- Scales clusters up and down even when jobs are running
- Builded by the second, 1 minute minimum
- Save money with preemptible instances
- Spark machine learning (MLlib) to run classification algos

### Dataflow

- If data is realtime or unpredictable
- Unified programming model, and service
- Batch and streaming data, same pipeline
- Clusters autosized, automatic scaling
- Optimized work partitioning, reduces worry about "hotkeys", lots of data getting mapped to same cluster

### Big Query

- AdHoc SQL on big data
- Data warehouse
- Cloud storage or Cloud datastore imports, or stream it at 100k rows a second
- Read and write BigQuery with Hadoop and Spark
- Global, with region storage
- You pay for data storage separate from queries
- Sharing data means customers pay for the query, not you
- Automatic discount for long-term storage

### PubSub

- stream analytics, decoupled send/receive
- support for offline consumers
- at least once delivery at low latency
- 1 million+ messages a second, and beyond!
- IOT systems

### DataLab

- Interactive Jupyter engine
- Orchestrates multiple GCP resources automatically
- Integrated with BigQuery, Compute Engine, and Cloud Storage

## Machine Learning

- Tensorflow -> Cloud ML
- Machine Learning APIs
