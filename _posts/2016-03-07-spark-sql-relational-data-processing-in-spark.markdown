---
published: true
title: Spark SQL: Relational Data Processing in Spark
layout: post
---
# Spark SQL: Relational Data Processing in Spark
Spark SQL is successor of Shark, SQL engine built on top of Spark. Spark SQL makes two contributions: (1) DataFrame (2) Catalyst.

## DataFrame

* Inspired from DataFrame in R, Python Pandas. But unlike R and Python, and like Spark, DataFrames are lazily evaluated lending the opportunity to optimize relational queries (e.g. computing multiple aggregates in one pass over data).
* Bridges the gap between two different models of computation in Spark: (1) procedural, and (2) declarative (e.g. SQL). Spark SQL allows seamless intermixing of procedural and declarative APIs.
* Collection of structured records (analogous to tables in RDBMS). Can be manipulated through common APIs of RDDs. Similar to RDDs, DataFrames are lazily evaluated i.e., unmaterialized and embodies a logical plan to construct them and it is possible to materialize DataFrames by calling "actions". During lazy evaluation, all the operations applied on DataFrames build up abstract syntax tree (AST). Before execution, the tree is passed to Catalyst to optimization. Logical plan of DataFrames is evaluated eagerly. In other words, if the user references a column which does not exist, the error is notified to user immediately not in the query optimization phase. This helps developers to immediately notice and debug simple errors.
* DataFrames support all basic SQL data-types. In addition, they support complex data types like structs, arrays. maps, union.
* Beside APIs, DataFrames also support native SQL syntax.
* Spark SQL can also cache DataFrames in memory in columnar format (column-major order) which is more compact than JVM/Python objects. Also columnar format enables efficient filtering and projection of rows.
* DataFrames also support User Defined Function (UDF). UDFs do not need special syntax or tools and can be created by simply lambda function in Scala.

## Catalyst
* Catalyst is an extensible query optimizer. User can add new optimization rule, new data source, data-source specific rules, and new data types.
* Catalyst optimizer is a rule-based optimizer. The AST is rewritten based on the rules until it reaches fixed-point - the tree stops changing upon applying the re-write rules. Logical optimizations include constant folding, predicate pushdown, projection pruning, null propagation, Boolean expression simplification, and other rules. Physical optimizations include pipelining projections or filters into one map operation, pushing operations from the logical plan into data sources that support predicate or projection pushdown.
* Catalyst generates Java bytecode (also type checked) which in turn transformed into native instructions in run-time by JVM JIT compiler to speed-up the query execution. Without native code generation AST has to be interpreted for each row of data, traversing AST and virtual function calls would make the execution very slow.
* __User Defined Types:__ Catalyst provide options to user for defining user defined data types that constructed from Catalyst built-in types. Built-in types are optimized to store in columnar compressed format, so user defined datatypes have the same efficiency of built-in types by simply composing from them. To define a new type, user has to provide conversions methods from object of that type to a Catalyst row and vice-versa

## Discussion
* Impala also optimizes and generates native code from query during run-time. Maybe that is one of the reason there is not much difference in performance between Spark SQL and Impala.

## References
