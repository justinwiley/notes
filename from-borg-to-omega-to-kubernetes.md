# Kubernetes


## Borg

[Borg was the pre-cursor](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43438.pdf) [to Kubernetes](https://kubernetes.io/blog/2015/04/borg-predecessor-to-kubernetes).  "It achieves high utilization by combining admission control, efficient task-packing, over-commitment, and machinesharing with process-level performance isolation."  Borg apparently [remains in-use at Google](https://www.quora.com/Does-Google-use-the-Open-Source-Kubernetes-or-a-version-of-Borg-for-their-own-container-management) for legacy systems.

### History

Borg grew out of 2 previous systems, ["Babysitter" and the "Global Work Queue"](https://queue.acm.org/detail.cfm?id=2898444).

Borg developed the concepts of:

- *tasks* - a single workload, possibly a single binary
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

### Services

Borg splits services into:

- *Long-running services* that are mission-critical, never fail applications like Gmail, Google Docs and the like.
- *Transient batch jobs* that take a few seconds to days to process

Borg was/is the workhorse technology behind:

- MapReduce
- FlumeJava
- Pregel
- Google File System
- CFS
- Bigtable
- Megastore

and many more!

The [white-paper](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/43438.pdf) suggests that multiple cells live in a cluster: "...The machines in a cell belong to a single cluster, defined by the high-performance datacenter-scale network fabric that
connects them." This points to Google's need to [virtualize networking on a large scale](google-cloud-networking.md) to support this sort of logical separation.  Especially given median cell size is quoted at "10 k machines after excluding test cells; some are much larger."

### Constraints -> Tasks -> Kubernetes

All of this fabric is heterogeneous in terms of memory, CPU, etc.  Borg jobs operate under constraints that allow the job to define what dimensions are requirements.

Borg jobs point to tasks, which in turn point to OS processes running in "containers".  These containers were presumably created before Docker and influenced it, then Kubernetes.

## Omega

Omega was driven ["by a desire to improve the software engineering of the Borg ecosystem."](https://queue.acm.org/detail.cfm?id=2898444)

- Paxos based, transaction oriented



