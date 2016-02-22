---
published: true
title: Paper Summary: Pregel: A System for Large-Scale Graph Processing
layout: post
---
Pregel: A System for Large-Scale Graph Processing
==========================================

* Developed at Google for distributed graph processing
* Bulk Synchronous Parallel (BSP) parallel computing model
* Distributed message passing system

Why Graph Computation Is Different
----------------------------------------------
* Poor locality of memory access
* Very little work has to be done per vertex
* Degree of parallelism changes substantially over the course of execution

Why not MapReduce
------------------------------
* Even though much data might be unchanged from iteration to iteration, the data must be reloaded and reprocessed at each iteration, wasting I/O, network bandwidth, and processor resources.
* The termination condition might involve the detection of when a fixed point is reached. The condition itself might require an extra MapReduce job on each iteration, again increasing resource use in terms of scheduling extra tasks, reading extra data from disk, and moving data across the network.

Bulk Synchronous Parallel (BSP) Model
--------------------------------------------------
* Computations are consist of a sequence of iterations, called superstep.
* Supersteps end with barrier synchronization
* During a superstep _S_, framework calls user-defined "Computation" function on every "logical worker" (in Pregel, a vertex) that works on _local_ (worker's own piece of) data in parallel.
* All communication is from superstep _S_ to superstep _S+1_.
* Ensure that programs are inherently free of classic concurrency bugs e.g. deadlocks, data-races. Performance of BSP model is predictable.

Pregel Model of Computation
---------------------------------------
* "Think like a Vertex" - vertex centric computation. No edge centric computations
* Algorithm terminates when all vertices vote to halt and there are no messages to deliver.
* Initially every vertex is active.
* All active vertices participate in the computation in a superstep. "Computation" function specifies behaviour at a single vertex _V_ and a single superstep _S_. During "Computation", a vertex can mutate state of its own and outgoing edges.
* At the end of each superstep a vertex can send message to other vertices in the graph. Messages are typically sent along outgoing edges. The messages are delivered at the beginning of next superstep. Message may be sent to any vertex whose identifier is known.
* A vertex deactivates itself by vote to halt. Deactivated vertices are not allowed to participate in computation. Vertices are reactivated upon receiving message.
* Graph state is kept in-memory, occasional saving checkpointing to disk for fault tolerance

Message Passing
---------------------
* Vertices communicate directly with one another by message.
* A vertex can send any number of messages.
* Each message consists of a message value, and the destination vertex.
* There is no guaranteed order of messages.
* Messages are guaranteed be delivered and not duplicated.

Master
---------
* Partitions the input graph and assigns one or more partitions to each worker.
* Responsible for coordinating the workers.
* Keeps list of all workers known to be alive, worker's unique identifiers, addressing informations and which partition of the graph is assigned to the worker.
* Size of data structure is proportional to the number of partitions not the number of vertices, number of edges.
* Maintains statistics of the progress of computation and the state of the graph.
* Runs HTTP server for user monitoring.

Worker
---------
* Maintains the current state of assigned partition(s) of the graph, responsible for computation of the assigned vertices, and delivers messages.

Message Combiner
------------
* Sending message incurs overhead which can be reduced in some cases.
* System calls "Combine()" for several messages intended for a vertex _V_ into a single message containing the combined message.
* User defined, application specific.
* Not enabled by default.
* No guarantee which messages will be combined, or in what order. Therefore, combiners should be enabled for commutative and associative operations.

Aggregator
--------------
* Provides mechanism for global communication, monitoring, and data.
* Each vertex can provide a value to aggregator in each superstep _S_, the system combines these values using a reduction operator, and resulting value is made available to all vertices at next superstep _S+1_.
* Only reduces input values from a single superstep.
* Possible to define a sticky aggregator that uses input values from all supersteps.
* Aggregator operation should be commutative and associative.

Fault Tolerance
--------------------
* Achieved through checkpointing.
* Upon failures, instead of re-computing all vertices from last checkpoint, only the lost partition is re-computed.

Graph Mutation
--------------------
* Input graph can be mutated in run-time.
* Mutations become effective in the superstep after the requests are issued.
* Within superstep, removals are performed first. All mutations are before "Computation()". First edge removal, vertex removal. Additions are after removal, first vertex addition, edge addition.
* Local mutations (mutating own edges) becomes immediately effective since there is no reason of conflicts.

Experiment
---------------
* Naive Single-Source-Shortest-Path was used
* The time for initializing the cluster, generating the test graphs in-memory, and verifying results is not included in the measurements
* Checkpointing was disabled
* Default hash partitioning was used

Critique
----------
* No fault tolerance for master is described in the paper
* How varying degree of parallelism i.e., load-balancing among workers over the course of execution is handled not mentioned
* How the message delivery is guaranteed is not mentioned
* Given the power-law distribution of real world networks, static hash partitioning is sub-optimal. This is is addressed in PowerGraph paper
