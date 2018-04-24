# Design and Process with Google Cloud

https://www.coursera.org/learn/cloud-infrastructure-design-process

This module covers building a cloud service, based on Google best-practices.

## Defining the Service

Defining a service involves starting with a rough design created by the whole team, then doing a more structured design process, and ultimately creating measurements to determine if we succeed with the service goals.

Everything at Google involves building:

1. The presentation layer (which they define as "networking"?)
2. Business logic layer ("compute")
3. Data layer ("storage")

*There are no universal solutions, only contextual solutions.*

Avoid *recency bias*, the old adage about change for change's sake.

### State

*State is any action in a system that depends on the memory from a proceeding action.*  Stateful vs stateless is a big impact on design, "state is the cornerstone of any design."

Stateful:

```
p1 -> p2 -> p3 (p1 happens before p2)
```

Stateless:

```
p1 p2 p3 (no dependency between processes)
```

#### Sources of State:

- Objects in a database
- Shared documents in memory
- User profile

#### Where is the truth in the system?

A central control mechanism is a choke point in the system, contributes to performance issues and can be a central point of failure.  State stored at these locations (for example shopping card in the ecommerce application).

*The best state is no state.*  But if you have to store state, what's the best place to store it?

#### Hotspots

If apps are co-located with state, and lots of users whose names start with a-f, load-balancing doesn't really balance, because requests pool on app server 1.

```

user request -> load balancers -> app server 1 (user a-f's profile) !!!
                               -> app server 2 (user f-r's profile)
                               -> app server 3 (user r-z's profile)
```

...Google suggests pushing state to a backend:

```

user request -> load balancers -> app server 1
                               -> app server 2 -> data store + cache
                               -> app server 3
```

But cache can become stale, so you can distribute the state with Cloud Load-balancing to replicated backend data stores.

```

user request -> load balancers -> app server 1                  -> data store 1
                               -> app server 2 -> load balancer -> data store 2
                               -> app server 3                  -> data store 3
```

This can introduce latency, but you pretty much have to live with it.

#### General Solution for Large-Scale Cloud Based Systems

```

stateless                     stateless                               stateful

dns + https                   many small    network                    sharded
loadbalancing  -> request ->  stateless  -> load balancer -> backend -> data
                              servers

```

This design allows you to control:

- Server discovery
- Request balancing
- Throttling

Using preemptible VMs means they can go down at any-point, and therefore must be stateless.  Adding them into your architecture can force you to think about where state has to live.

### Measurements

Measurement helps you to:

- Make design choices
- Test and validated
- Monitor during operation

In the beginning, you can't measure, so estimate. Actual numbers don't matter, orders of magnitude matter do, 10 vs 100 vs 1000.

#### Defining Measurements

- Service level objectives (SLOs)
    + Quality of service or attribute to measure, what is the effect on the customer
    + They can be high level "user can see the shopping cart", or low-level "should be available 99.9% of the time"
- Service level indicators (SLIs)
    + How you measure the SLOs
    + "How do you measure 99.9% availability": "back-end server up 99.9% of the time"
    + Max 95%, min availability, min freshness etc
- Service level agreements (SLAs)
    + How are the KOs maintained?
    + What happens when the service is outside the KOs?
    + Communications to users?
    + Consequences?

#### SLIs

Requirement: user is responsive to users
Objectives: no more than 1/2 second latency 
Indicator: round-trip time

| SLO | SLI |
| ------------- | ------------- |
| Performance | Latency, throughput (qps), offered load, load shedding, velocity, obtainability, freshness |
| Availability | Uptime, outage frequency / duration, reliability |
| Quality | Accuracy, correctness, completeness, coverage, relevance, security |
Internal State | Queue Length, memory use |
People | Time to respond, time to fix, fraction fixed correctly |

#### Real KIs beat Ideal

| Indicators | Impossibilities |
| ------------- | ------------- |
| Error budget | ~~Error free~~ |
| Unplanned downtime | ~~Zero downtime~~ |
| Risk tolerance | ~~Zero risk~~ |
| Target availability | ~~Always available~~ |

Also: SLIs should be useful enough to warrant a human be involved, very expensive
1. Numerical

#### Create actionable alerts

- Understand you users
    + Some are far away, and care about round-trip times
- Understand the context
    + Do you want sales from the last 5 minutes or the last 3 years

### Requirements

*Design always starts with question gathering.*

#### 1. Qualitative Requirements

|  |  |
| ------------- | ------------- |
| Why | Why is the system needed?  Why a new landing pad? |
| Who | Who are the stakeholders, users, staff needed |
| What | What does it need to do specifically? |
| When | When?  Do you want quality or speed? |

#### 2. Qualitative Requirements

- Time
    + Operational time-constraints
    + Cost of downtime
- Data
    + Cost of data lost
    + Data volume
    + Throughput
    + Freshness
    + Groups of data
- Users
    + Number
    + Location
    + Demographics

#### 3. Scaling

- Growth requirements
    + How does storage, CPU, network increase?

#### 4. Size Requirements

- Dimensions
- Replication
- Rate of change

## Example App - Thumbnail photo service

Google develops a sample application, a service where users can upload an image, get back a thumbnail.

### Workflow and First Design

```
Input -> Image Storage -> Thumbnail Conversion -> Thumbnail Storage -> Thumbnails
```

First Design:

Single App Server -> Log Data

### Gather Requirements

- users: on the internet, using a web browser, want to upload images and get a thumbnail in return
- speed: < 1 minute, beating estimate thumbnail time on computer
- resources: we don't know make a guess
- scale: this information will be given during the design exercise
- size: application can be served to all users from one location, say central-US to start
- availability: loose, no-one relies on it.  Most of the time

### Define Service Level Objectives (SLOs)

- availability: 23 hours a day, 95.83%

#### Tests

- Pre-production tests
- Unit-tests
- Integration tests
- System tests
- Stress tests
- Roll-out

## Business Logic Layer Design

Part of the project that encode the real-world business rules that determine how data can be created, stored and changed.  For example: A work-flow of purchasing a ticket can occur through multiple front ends, but the process stays the way.

### Micro-services

Atomic, single purpose(ish) services

#### Benefits of small services:

- Easier to develop and maintain
- Does one thing well
- Supports a/b testing

#### Independently developed serviecs aid in

- Fault isolation
- Debugging
- Redundancy and resiliency

#### Downsides

- It's harder to understand how micro-services interoperate
- Unit-testing is easier, integration testing is harder

#### Micro-services on Google Cloud

- Cloud Functions
    + Latency, one language
- App Engine
    + No local file system
    + One master app per project

#### 12-factor on Google Cloud

1. One code-base tracked in version control many deployments
    - Cloud Shell for deploying and building
    - Cloud Source Repositories / Github
2. Strict separation between build and run phases
    - Build GAE (Google App Engine) in Cloudshell, upload to GAE
    - You can simulate all that locally on your desktop for integration testing
3. Keep development, staging, production as similar as possible
    - Deployment manager templates, consistent build process
4. Explicitly declare dependencies
    - Custom images, with clearly defined libraries, environment etc
5. Store config in environment
    - Difficult, because you don't want to stick credentials into ode
    - So you can use Google Metadata server, or store in Google Cloud Storage
6. Maximize robustness with fast startup and graceful shutdown
    - Instance templates create the same instance every time, managed instance groups use the same startup scripts etc, auto-scaling

### Which Platform?

 - Think about AppEngine first
     - SnapChat built on AppEngine
     - Code first
     - Minimize overhead
     - Autoscale
     - Can run on GAE Flex
 - Google Kubernetes
     + If you need more knobs and switches go to Kubernetes
     + If you need cloud independence
     + No OS dependencies
 - Compute Engine
     + HPC
     + Processing efficiency

#### How to grow the Thumbnail Service?

- Vertical scaling (up to 64 cores)
    + If we scale one box its easier
    + But it's a single point of failure
- Horizontal Scaling
    + The ideal
    + Automation easy, rolling deployments
    + More overhead
    + E2E latency increases somewhat, more network chatter
    + Outweighed by decoupling scaling, failures, upgrades, config

#### Horizontal How-To

- Keep servers simple
    + Minimize complexity
    + Design simple APIs
    + Identify where tasks are separable
    + Split into separate services
- Prefer smaller, stateless servers
    + Easy to scale, no state to shard / rebalance
    + Failure is cheap, no state to manage or recover
    + Easy to load balance, no hot-spotting

Does horizontal scale just mean two boxes? *Google suggests:*

**Number of Queries per Second / 3**

With 3 instances of the same service, you have redundancy without too much overhead.  If you just have 2, and one fails, the remaining one has to absorb 100% of the workload.

| Small Stateless Servers Increase Reliability and Scalability | Large Stateful Servers Reduce Complexity and Latency |
| ------------- | ------------- |
| Divide into parts | Unify |
| Duplicate and Coordinate | Simplify and Consolidate |
| Seperate and Isolate | Coalesce and co-locate |

Methods of achieving balance:

- What are your SLOs?  What do your users value?
- What is the optimal size and number of parts?  1 service, 10 microservices, 100?
- Sometimes central control is optimal
- Plan on adjusting, build adjustment process

#### Recommendations

- Use where there are many consumers of atomic functionality
- When there is one consumer of tightly coupled functionality, micro-services could be too much overhead

### Thumbnail Service is Slowing down

What to do about it?

Systematic SRE troubleshooting:

1. Segment and reduce the problem space
    - Where can slow-downs affect the system
2. Step through the system manually in your mind
3. Add more monitoring and logging

#### Best Practices

- Five why's
- People are never the root cause, behaviors are
    + Blaming people leads to fixing the wrong things

```
Input -> Image Storage -> Thumbnail Conversion -> Thumbnail Storage -> Thumbnails
```

Thumbnail conversion looks like it might be the culprit, but it could be:

User Experience
- HTTP server
- Dynamic / static content
- Session handling

Ingest
- HTTP
- High throughput
- Low disk I/O

Thumbnail Conversion
- High CPU
- Memory
- Low disk/io

Image Storage
- Large files
- Small files
- Many files
- High r/w

Serving thumbnails
- HTTP
- high read I/O
- Dynamic / static content
- Session handling

After reflecting, the lecturer decided thumbnail conversion was the culprit.

Business issue: User experience is being impacted by the thumbnail conversion process

#### Possible Solution - Split Monolith to Offload CPU

- Offload high CPU process to another service
    - This will reduce memory allocation on web-server
    - Reduce some disk I/O
    - Moved image storage to thumbnail processor

**After implementing, we check our indicators to see if the approach was successful**

| Objectives | Indicators |
| ------------- | ------------- |
| *Before: * - Availability, 23/24 hours a day = 95.83% | Aggregated server up/down time |
| *After: * - Utilization of thumbnail server | CPU untilization 48% |


## Data Layer Design

## Presentation Layer Design

## Design for Resiliency, Scalability, and Disaster Recovery

## Design for Security

## Capacity Planning

## Deployment, Monitoring and Alerting, and Incident Response
