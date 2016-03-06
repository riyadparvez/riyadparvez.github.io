---
published: true
title: Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing
layout: post
---
Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing
===============================================

The Case Against MapReduce
------------------------------------------
* Many applications (almost all machine learning algorithms) are iterative. Analysis is essentially a sequence of "iteration" where the result/output of previous iteration is used as input for next iteration.
* The MR way of computing iterative algorithms is series of MR job. After each MR job, intermediate results are saved in disk and then loaded again for the next iteration MR job. Although from fault tolerance perspective, this is beneficial because all intermediate results are stored in disk; so if a machine fails, only the work in done in current iteration is lost. But the cost of storing all intermediate data in disk outweighs the benefit of fault tolerance which occurs infrequently in practice.

Resilient Distributed Datasets (RDDs)
--------------------------------------------
* RDDs are distributed parallel data structures that can be partitioned among different machines, provide fault tolerance. RDDs provide rich set of high level APIs to programmer; Spark authors argue that these APIs often enough to express computation of large number of application and more expressive than existing frameworks like MR, HaLoop.
* RDD can be created two ways: (1) from a data source (2) from other RDDs by transformations.
* Computation on RDD is divided into two types: (1) transformations (2) actions. Transformations (e.g. map, join, filter) on RDDs are lazily evaluated; Spark keeps the lineage (log) of a RDD that how it is derived from other RDDs. Lineage also provides fault tolerance for RDDs, because if a RDD is lost in case of failure, the information for recomputing the RDD is already available. Actions (e.g. collect, take, count) are evaluated eagerly. In other words, Spark only starts computation when it encounters "actions"; otherwise Spark keeps lineage of RDDs as in lazy evaluation manner.
* Lazy evaluation in Spark is essential for setting up the data pipeline, setting up directed acyclic graph (DAG) of a job where nodes represent computation to be performed and edges represent data-flow between tasks. DAG model of data pipeline was first introduced in Dryad [1].
* In case of failures, only the lost partitions of RDDs will be recomputed from the lineage information of RDDs. Unlike other frameworks, where the whole application has to be rolled back to the last checkpoint, Spark only recomputes the lost paritions.
* For iterative applications, online analytics, intermediate results i.e., RDDs can be 'persisted' in memory, disk. Users can also prioritize specify which RDDs should be spilled to disk first.

Discussion
--------------
* RDD concept is interesting because now fault tolerance is not a property of a job, but it is a property of RDD. Computations are expressed in functions of RDD, so for computations (transformation of RDD) that are very expensive, Spark provides an API to checkpoint those RDDs instead of checkpointing the whole state of a job.
* The design choice of Spark is interesting. One of the assumption behind Spark was opposite of mantra popular in that time. For fault-tolerance, most distributed frameworks checkpoints i.e., stores intermediate results into the disk, even in case of failures of task, computation is not lost. In other words, computation i.e., CPU time is expensive relative to the time spent in checkpointing i.e., disk IO. Spark went against this mantra assuming disk IO is more expensive than CPU time. Therefore, it is better to just recompute the whole computation in case of failure instead of checkpointing periodically. Although a recent paper [2] and Project Tungsten of Spark showed that CPU is the bottleneck for Spark not disk or network. But this does not invalidate Spark assumption. The CPU bottleneck is due to serialization, deserialization of JVM objects, virtual method calling which would not make a difference even if we switch to classic checkpointing approach.


References
--------------
[1] [Dryad](http://research.microsoft.com/en-us/projects/dryad/)

[2] [Making Sense of Performance in Data Analytics Frameworks](https://www.usenix.org/conference/nsdi15/technical-sessions/presentation/ousterhout)
