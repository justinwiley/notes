# From Borg To Omega To Kubernetes

To understand what Kubernetes is, it's useful to understand where it came from, what problems it was designed to solve and how earlier systems tried to tackled those same problems.  Kubernetes started it's life in Google, and was inspired by (at least) two earlier job orchestration projects.  Chief among these was Borg, and Borg's successor, Omega.

I'm basing this analysis off of my interpretation publicly available sources.  I annotate anything I fail to fully understand, or where there is limited source material with a "*?*".

## Cluster schedulers

All of these frameworks are [*cluster schedulers*](https://medium.com/@copyconstruct/schedulers-kubernetes-and-nomad-b0f2e14a896).

The need for cluster schedulers evolved along with the need for massive data-processing centers, and the need to efficiently [manage huge computing resources](https://www.morganclaypool.com/doi/pdf/10.2200/S00193ED1V01Y200905CAC006).

The concept of a job, extrapolated from earlier work on Grid computing and the [long history of distributed computing](https://en.wikipedia.org/wiki/Computer_cluster) and job scheduling at the OS level was a key part to this.

Cluster schedulers had to be designed in the context of:

- *Workload churn* - constantly changing jobs and resource requirements
- A *heterogeneous platform* - jobs needed to run on different resource types and capabilities
- Encouraging *paralleism* of similar workloads.  [MapReduce](https://research.google.com/archive/mapreduce.html?hl=de) and [Pregel](https://blog.acolyer.org/2015/05/26/pregel-a-system-for-large-scale-graph-processing/) required a reliable, scalable substrate to execute on
- *Resilience*

*Interesting question: what's the difference between a distributed cluster scheduler, and a distributed database?*

## Borg

Borg grew out of 2 previous systems, ["Babysitter" and the "Global Work Queue"](https://queue.acm.org/detail.cfm?id=2898444).  The Global Worker Queue was particularly important, although it was focused [primarily on batch jobs](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44843.pdf).

[Borg is considered to be the direct predecessor to Kubernetes](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43438.pdf) [to Kubernetes](https://kubernetes.io/blog/2015/04/borg-predecessor-to-kubernetes).  "It achieves high utilization by combining admission control, efficient task-packing, over-commitment, and machine-sharing with process-level performance isolation."  Borg apparently [remains in-use at Google](https://www.quora.com/Does-Google-use-the-Open-Source-Kubernetes-or-a-version-of-Borg-for-their-own-container-management) for legacy systems.

Borg was/is the workhorse technology behind:

- MapReduce
- FlumeJava
- Pregel
- Google File System
- CFS
- Bigtable
- Megastore

and many more.

### Overview

Borg developed the concepts of:

- *tasks* - a single workload, or a single container
- *jobs* - a collection of tasks
- and a *cell*, a set of machine managed as a unit

Borg also relies on

- a *Borgmaster*, consisting of:
    + A main process
    + Scheduler
    + 5 replicas, containing information on the state of a cell
    + Paxos leader election strategy for organization
    + Check-pointing of state
- and "*Borglets*"
    + local Borg agent on each cell
    + Starts and stops tasks
    + Maintenance, monitoring, log-rotation, etc

### Jobs

A job is a "[complex vector](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44843.pdf) of one or more identical tasks, indexed sequentially from zero."

### Allocs

Allocs are reserved set of resources on a (single?) machine.  Tasks run inside of allocs, and *alloc sets* are considered to be similar to a job.

How are allocs constructed and defined?  It's a little unclear, but some sources suggest chroot jails and early containers:

["We use a Linux chroot jail as the primary security isolation mechanism between multiple tasks on the same machine...VMs and security sandboxing techniques are used to run external software by Google’s AppEngine."](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43438.pdf)..."Early versions of Borglet had relatively primitive resource isolation enforcement," but eventually this evolved to all "tasks being run inside a Linux cgroup-based resource container".

It's interesting to note that the Wikipedia article on [Cgroups](https://en.wikipedia.org/wiki/Cgroups) says: "Engineers at Google (primarily Paul Menage and Rohit Seth) started the work on this feature in 2006 under the name 'process containers'".  This work apparently predated Borg, but was perhaps considered to be too immature for the initial roll-out of Borg.  If containers had been widely used and well-understood, it's possible Borg would have resembled Kubernetes more closely. 

### Services

Borg splits services into:

- *Long-running services* that are mission-critical, never fail applications like Gmail, Google Docs and the like.
- *Transient batch jobs* that take a few seconds to days to process

The [white-paper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43438.pdf) suggests that multiple cells live in a cluster: "...The machines in a cell belong to a single cluster, defined by the high-performance datacenter-scale network fabric that
connects them." This points to Google's need to [virtualize networking on a large scale](google-cloud-networking.md) to support this sort of logical separation.  Especially given median cell size is quoted at "10 k machines after excluding test cells; some are much larger."

### Constraints -> Tasks -> Kubernetes

All of this fabric is heterogeneous in terms of memory, CPU, etc.  Borg jobs operate under constraints that allow the job to define what dimensions are requirements.

Borg jobs point to tasks, which in turn point to OS processes running in "containers".  These containers were presumably created before Docker and influenced it, then Kubernetes.

### Lessons from Borg

#### "Orchestration is the beginning, not the end"

"As more and more applications were developed to run
on top of Borg, our application and infrastructure teams
developed a broad ecosystem of tools and services for
it...the result was a somewhat heterogeneous, ad-hoc collection of systems that Borg’s users had to configure and interact with."

- https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44843.pdf

Some of these services included:

- Naming and service discovery
- Master election
- Application load balancing
- Rollout tools
- Workflow tools
- Monitoring tools


- Jobs were the wrong unit for abstraction
    + No way to manage complex multi-task jobs
    + No way to identify and manage related jobs (production versions and development versions for example)
    + Led to the idea of Pods
- Network resouce grouping as important as process grouping
    + Kubernetes can take advantage of improvements in VMs, IPv6, and [Linux namespaces](https://en.wikipedia.org/wiki/Linux_namespaces)
- Allocs -> Pods
    + Grouping resources is useful, pods extended that idea
- Introspection and monitoring are key

## Omega

[Omega](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/41684.pdf) was another offspring of Borg, was driven by a desire to improve the software.  Omega was actually publicly discussed [before Borg or Kubernetes](https://www.nextplatform.com/2016/03/22/decade-container-control-google/), but was developed after Borg, and co-evolved (?) with Kubernetes and was influenced by it / influenced it.  "Many of Omega's innovations (including multiple schedulers) have since been folded into Borg".

Omega is a cluster scheduler designed with the following constraints in-mind:

- High resource utilization
- User-supplied placement
constraints
- Rapid decision making
- Various degrees of “fairness” and business importance
- Robust and always available

### Omega was driven ["by a desire to improve the software engineering of the Borg ecosystem."](https://queue.acm.org/detail.cfm?id=2898444)

- More "consistent, principled architecture"
- Paxos based, transaction oriented
- Multi-master, splits Borgmaster into multiple decoupled components
- Exposes components directly to trusted components
    + As a database?  Presumably external services can interact with it directly
    + Kubernetes wraps these under a consistent API

Earlier takes on cluster management were based around two approaches:

- A single monolithic scheduler
    + Hard to add new policies
    + "Don't scale to large cluster sizes"? (Must be pretty large)
- Or two-level scheduling
    + A single active resource manager that allocates resources, and then chooses a resource framework to hand off those resources to
    + [Mesos](http://mesos.apache.org/) and [Hadoop On-Demand](https://hadoop.apache.org/docs/r1.2.1/hod_scheduler.html)
    + In-practice they hide resource visibility and aren't flexible enough to schedule more complex jobs
    + Over time it becomes increasingly difficult to add new policies

Omega is built around a parallel scheduler and *shared-state*, and lock-free optimistic concurrency.

- Each scheduler has full access to the entire cluster, competing with other schedules
- Contention is solved via optimistic concurrency control

### Still in use?

The most recent [publicly available article](https://www.nextplatform.com/2016/03/22/decade-container-control-google/) on Omega was written in 2016, so it's unclear how widely Omega is used, and if it ever made it past the planning stages.

One of Omega's strengths compared to Borg and presumably Kubernetes is it's ability handle massive cell-sizes.  If possible that use- case is no longer as pressing to Google, possibly due to better isolation between services, or easier mapping to Kubernetes primitives.

## Kubernetes

"Kubernetes is, first and foremost, a pod management system, keeping the container collectives aligned and running."

Kubernetes was developed along with Omega, and was publicly announced in 2014.  1.0 was released in 2015, and became an open-source project.

While Kubernetes is probably simpler, or at least more obviously constructed than Borg, it would be an exaggeration to call it simple.  Understanding all of the primitives, how they collaborate, and how to operationally run Kubernetes and deal with details like [backing up etcd's data](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/) is not a trivial undertaking.

### Philosophy

"The design of Kubernetes as a combination of micro-services and small control loops is an example of control through choreography—achieving a desired emergent behavior by combining the effects of separate, autonomous entities that collaborate."

- https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44843.pdf

Kubernetes fundamentally exists to define rules for the game of dividing scarce resources among tasks.  Kubernetes' designers fundamentally agreed with Omega's approach of avoiding monolithic, top-down schedulers, but refined it's consistency primitives ([moving to etc and Raft from Paxos?](https://medium.com/containermind/a-new-era-of-container-cluster-management-with-kubernetes-cd0b804e1409)) to be more transparent.

["Kubernetes has implemented almost all the container cluster management features of Omega and added more advanced features with best-of-breed ideas and practices from the community."](https://medium.com/containermind/a-new-era-of-container-cluster-management-with-kubernetes-cd0b804e1409)  Kubernetes seems to rely heavily on CoreOS technologies like [etcd](https://github.com/coreos/etcd) and [flannel](https://github.com/coreos/flannel), embracing a strategy of cherry-picking mature open-source technologies and corralling them into a coherent framework.

Kubernetes exists to bring the objects it has been asked to create to life, to control them, scale them, and ultimately reap them when their time has come and release their resources to new processes.

Kubernetes embraces the master-slave pattern, and a [control-plane](https://en.wikipedia.org/wiki/Control_plane) abstraction.

### Pods

Jobs in Borg were vectors of tasks.  If a task died or completed, the vector became sparse, and individual tasks couldn't be managed or re-evaluated.

Kubernetes introduces the concept of a *pod*, which is it's key unit of scheduling.  A pod is a resource envelop around a collection of containers, storage and a shared (virtual) network.  A pod has a *specification* that determines how it will be executed.

Pods make it dramatically easier to launch collections of containers together, mitigating the "one process per container" principle.  This includes side-car containers and proxies for monitoring individual containers.

#### Pods come and go

["Pods are mortal."](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/)  Pods follow a defined life-cycle:

- Pending
- Running
- Starting
- Succeeded
- Failed
- Unknown

#### Communication between Pods

Containers within a pod share an IP address and port space, and so they can communicate with each other via localhost, or even via IPC.  Pods can share storage space, so two containers can write to a shared log file.

## Coordination and etcd

"Clusters are usually built from a large collection of machines with the ability to run any workload at any given time. In order for a cluster to perform at high levels of efficiency, we need to distribute workloads appropriately across all machines in the cluster.  Then clusters need a way of coordinating with each other.

For example a job scheduler needs to notify a machine that it has work to do. Once that work has been completed machines may need to communicate that fact to some other component in the cluster. A distributed system needs a reliable coordination mechanism, so it’s important that this communication happen in a timely and reliable manner to keep everything running smoothly. In essence something has to manage the state of the cluster – the source of truth.

This is where etcd comes in."

- https://thenewstack.io/about-etcd-the-distributed-key-value-store-used-for-kubernetes-googles-cluster-container-manager/

etcd is the core system for coordination in Kubernetes

- https://mysqlhighavailability.com/good-leaders-are-game-changers-raft-paxos/

## Nodes

A node (also "worker" or "minion") is a "machine" (bare-metal? VM more likely?) where workers are deployed.  A node runs a container runtime like Docker.  Containers live on nodes.

### Kublets

A kublet is the direct descendant of a borglet.  Kubernets -> *kub*let, Borg -> *borg*let.  Kublet's serve the same purpose, starting and stopping and controlling pods as directed by the control plane.

### Kube Proxy

A network proxy and load-balancer that sits on the node.  Consumers of the pods services (HTTP requests for example), are channeled through the proxy.  Containers can expose external IPs directly through.

### cAdvisor

A monitoring agent that lives on nodes.

### Services

Borg's purpose as a cluster scheduler is to ["manage the lifecycles of tasks and machines"](https://kubernetes.io/blog/2015/04/borg-predecessor-to-kubernetes), but to carry this out, the Borg teams had to construct a whole range of applications, including naming and load-balancing.  

Kubernetes uses a service abstraction to encapsulate these: a service has a name and maps to a dynamic set of pods defined by a label selector.

["A Service in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service."](https://kubernetes.io/docs/tutorials/kubernetes-basics/expose-intro/).

The set of pods in a service are usually defined by a label-selector. Obvious examples of 

#### Service Catalog

Kubernetes can wrap other services, for example a message bus, and manage them as though they were a part of an existing service. (?)  Possibly that extends to 

### Controllers

Controllers are a concept that encapsulate the actual orchestration of pods within a cluster.  Controllers are ["a reconciliation loop that drive actual cluster state towards a desired configuration."](https://en.wikipedia.org/wiki/Kubernetes)

There isn't one controller that does it all.  There are a few different kinds of controllers:

- *Replication Controllers* - Scales and replicates pods across a cluster
- *DaemonSet controllers* - Ensures only one pod runs on each machine (?)
- *Job Controllers* - Runs pods to completion, for batch jobs

### Labels and Label Selectors

Labels selectors also address the lack of fidelity in a Borg task.  Deleting a particular or modifying a pod can be accomplish using selectors, instead of using a task.

A pod might have the labels "*role=frontend*" and "*stage=production*",
indicating that this container is serving as a production
front-end instance.

Labels can be used for a variety of purposes, for example load balancing, and for triaging failed pods.  For example, if a pod fails, all labels can be removed from it, and it will stop being used for whatever purpose it had been used for (i.e. if load balancing, no more traffic will be routed to it), but it remains in place for analysis.

*Label selectors* are used to construct arbitrary groups of objects.  For example: stage==production && role==frontend.  Labels can be included in multiple sets.

### API

"Kubernetes attempts to avert this increased complexity by adopting a consistent approach to its APIs. For example, system evolution every Kubernetes object has three basic fields in its description: ObjectMetadata, Specification (or Spec), and Status. "

- https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44843.pdf

- *Object metadeta* - ["persistent entities in the Kubernetes system"](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/) object name, UID, version, labels
- *Specification or Spec* - the desired state of the object
- *Status* - read-only information about current state of the objects

https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/44843.pdf

API endpoints generally accept object specifications, and upon receiving them, work to make the system achieve the designed state.

### Objects and Specifications

Objects can describe:

- What applications are running and on which nodes
- Resources available to those applications
- Policies around how they should behave, like restart, upgrade and fault-tolerance policies.

"A Kubernetes object is a “record of intent”–once you create the object, the Kubernetes system will constantly work to ensure that object exists. By creating an object, you’re effectively telling the Kubernetes system what you want your cluster’s workload to look like; this is your cluster’s desired state."

- https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/

This is extremely similar to a popular quote about another task management system: ["...[the] terminator is out there. It can’t be bargained with. It can’t be reasoned with. It doesn’t feel pity, or remorse, or fear. And it absolutely will not stop, ever."](http://quotegeek.com/quotes-from-movies/the-terminator/1136/)  This leads me to feel we should trust Kubernetes to accomplish it's tasks, but not to trust that it always operates in our best interests.

Kubernetes objects are defined in specifications, YAML text files:

```
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Specifications resemble [CloudFormation templates](https://aws.amazon.com/cloudformation/aws-cloudformation-templates/), but seem to be simultaneously more abstract (generic objects vs defined AWS services), and more fine-grained (labels).

Specifications must contain the following fields:

- *apiVerion* - of Kubernetes API
- *type* - the type of object
- *metaData* - about the object
- *spec* - specifics about the object, which varies depending on the type

Kubernetes accepts specs via it's API, or directly from kubectl.

```
kubectl create -f https://k8s.io/docs/user-guide/nginx-deployment.yaml --record
```