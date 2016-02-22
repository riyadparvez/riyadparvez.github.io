---
published: true
title: One Trillion Edges: Graph Processing at Facebook-Scale
layout: post
---
One Trillion Edges: Graph Processing at Facebook-Scale
===============================================

Background
--------------
* Apache Giraph has started as open source implementation of Google Pregel [2]. I have covered the Pregel paper [here](). If you are new to distributed graph processing systems, please first cover the summary of Pregel.
* In this review, I will discuss Facebook experience and modification computation abstraction of Pregel model. Implementation details like  parallelization support in nodes to enable more local parallel computation, memory optimization is left out.

Sharded Aggregators
---------------------
* Aggregators in Giraph is implemented using Zookeeper znode (a type of data storage in Zookeeper). But Zookeeper has inherent limitation of maximum one megabytes znode. To circumvent this, workers can directly send values to master, but the master becomes the bottleneck. Therefore, "sharded aggregator" is introduced where workers are assigned randomly in shards. A worker becomes the leader of shards and compute the aggregator of its assigned shards. Now aggregation is balanced between master and the shared leaders.

Master and Worker Computation
---------------------
* "The methods _preSuperstep()_, _postSuperstep()_, _preApplication()_, and _postApplication()_ were added to the Computation class and have access to the worker state." - It seems very application (k-means clustering) specific. I am skeptical how useful these methods are.
* In Pregel, the Master abstains from actual graph processing, instead focuses on global coordination of workers.
* Facebook modified Giraph to introduce computation in master node. Master node computes central, and global computation and the result of this is available via aggregators to all workers in the following superstep.
* This model is very useful for lightweight task in global scope i.e., applies to all worker nodes. One simple example each iteration-bounded PageRank where the workers check the number of supersteps performed and "vote to halt" if desired number of computation is completed. This check can be easily performed in master after each superstep removing the complexities to check in every worker.

Composable Computation
------------------------
* Many graph application perform different type of computation each implementing different program logic in different stages. One "Computation" per vertex model in Pregel leads to code-bloat (a switch statement with different type of computation depending on the stage) in vertex computation. Giraph separates the computation itself from the vertex, computation is defined in separate class and the vertex can choose which type computation to perform at current superstep.
* Pregel model allows only message and message combiner type. Giraph splits this into two different message type: incoming message type and outgoing message type for each computation class. Giraph ensures types of computation, messages and combiners match at run-time.

Superstep Splitting
---------------------
* In some graph applications, vertices can send messages in pattern that exceeds the memory of a worker. Messages that are aggregatable i.e., commutative and associative, combiners solve this problem by combining multiple messages into one message. But some messages are not 
* In case of such message patterns, a "logical" superstep is split into multiple supersteps. A vertex sends only a fraction of all messages, and the receiving vertices can partially compute its state based on the received messages. Similar to aggregators, vertex update computation must be commutative and associative.

Discussion
--------------
* Varying degree of parallelism in graph processing systems is a huge issue. Simple graph computation like PageRank converges slowly because only a small sub-graph still has not converged yet. Stanford GPS [4] tries to address the issue of slow convergence in several graph algorithms.
* IBM has introduced "Think Like a Graph" [5] abstraction for graph processing. I am curious the centralized computation in master compares to this paradigm.

[1] [One Trillion Edges: Graph Processing at Facebook-Scale](http://www.vldb.org/pvldb/vol8/p1804-ching.pdf)

[2] [Pregel](http://googleresearch.blogspot.ca/2009/06/large-scale-graph-computing-at-google.html)

[3] [Apache Giraph](http://giraph.apache.org/)

[4] [GPS](http://infolab.stanford.edu/gps/)

[5] [From "Think Like a Vertex" to "Think Like a Graph"](http://www.vldb.org/pvldb/vol7/p193-tian.pdf)
