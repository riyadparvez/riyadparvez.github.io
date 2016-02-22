---
published: true
title: Mesos: A Platform for Fine-Grained Resource Sharing in the Data Center
layout: post
---
Mesos: A Platform for Fine-Grained Resource Sharing in the Data Center
========================================

Goal
-----
* Sharing a cluster between multiple cluster computing frameworks like YARN, MPI.
* To share a cluster among multiple frameworks, statically partition the cluster into sub-clusters and assign each framework a sub-cluster for computation. It is obvious how static partitioning leads to cluster under-utilization or overloaded. Other way is to introduce another layer of VM (e.g. AWS) and dynamically partition them. It increases complexity to manage another layer of VMs and inefficient.
* Frameworks like YARN, Dryad employ fine grained resource sharing model. But there is no way to share fine grained resource across the frameworks.

Overview
---------
* Mesos is a thin resource sharing layer that enables fine-grained sharing across diverse cluster computing frameworks. It provides client frameworks a common interface for accessing cluster resources. 
* Like YARN, Mesos is a resource negotiator for frameworks. Each framework is responsible for scheduling its own allocated resource. Advantage is each framework can schedule to its domain specific workloads instead of one-size-fits-all scheduler for all workloads. This is known as "two-level scheduling".
* Mesos offers resources to the frameworks in "resource offer". A resource offer is a bundle of resources that a framework can allocate on a cluster node to run tasks. Mesos decides how many resources to offer each framework, based on an organizational policy such as fair sharing, while frameworks decide which resources to accept and which tasks to run on them.
* Many frameworks are optimized for data locality - tasks are executed in the node where data is located. A framework can reject resource offers if the offer does not satisfy the constraint of data-locality and so on. Delay scheduling can be utilized where a framework waits for limited time to acquire resources on nodes storing the data. This is effective because most of the tasks in Mesos are short-lived.

Allocation Modules
---------------------
* Resource allocation decisions are provided by pluggable allocation modules (e.g. fair sharing, priority sharing) in Mesos. An organization can roll out its own resource allocation policy without need to change Mesos itself.
* Allocation can be revoked if tasks do not complete in short period of time since Mesos strictly focuses short-lived tasks. Mesos regains resources from long running tasks by killing them. Before killing a task, Mesos gives the framework a grace period to save its state or clean-up.
* Allocation modules decide whether resources should be revoked from frameworks since resource revocation heavily depends on organizational policy.
* Guaranteed allocation: Killing tasks has negligible impact on framework like Hadoop which is designed keeping fault tolerance in mind. But for frameworks like MPI, killing tasks can be fatal since tasks are highly synchronized and inter-dependent. For these frameworks, Mesos introduces "guaranteed allocation" - allocation offer that a framework can use without losing tasks. Tasks are never killed if the job utilizes resources below guaranteed allocation. One or more tasks can be killed if resource usage goes above guaranteed allocation.

Isolation
---------------------
* Isolates framework resources using existing OS containers e.g. Linux containers, Solaris jails. 

Fault Tolerance
---------------------
* Soft State: Master can recompute its internal state from messages from slaves and framework schedulers.
* Hot-standby: Masters is shadowed by several hot standbys. Upon current master's failure, Zookeeper selects a new master from the standbys. New master constructs internal state from the messages.
* Mesos also reports task, executor failures to the frameworks' scheduler. 
* What if the Framework Master/Scheduler fails: Mesos allows each framework to register multiple backup scheduler. When the current scheduler fails, Mesos notifies another scheduler to take over. Frameworks are responsible for sharing states between their schedulers.

Discussion
-------------
* Mesos is designed for frameworks for short-lived jobs e.g. batch analytics. But cluster is still statically partitioned between log-running jobs (e.g. web-servers, database servers) and batch analytics. It is possible to go one step above and also handle services and batch analytics in same resource negotiator.
* Mesos is pre-emptive that means it prevents conflict over resources beforehand. But study indicate that users of frameworks hence the frameworks themselves widely over-estimates resource needed for finishing a job i.e., users claim more resources than actually needed for finishing a job. This leads to servers under-utilization. Google Omega Scheduler [3] solves this problem by optimistic resource allocation i.e., more resources are promised to frameworks than actually available. Conflicts are resolved only after conflicts over resource actually occurs.

References
--------------
[1] http://hadoop.apache.org/

[2] http://research.microsoft.com/en-us/projects/dryad/

[3] http://research.google.com/pubs/pub41684.html
