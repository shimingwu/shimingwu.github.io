---
title: BigData(5)
tags:
  - Big Data
date: 2020-11-09 20:35:18
---



This is revision note for NUS master study of CS5425 Big Data Systems for Data Science.

## Hadoop VS Databases

>Apache Hadoop is not a database storage or relational storage, its main competency is to process data in a distributed fashion. It does have a storage component called HDFS (Hadoop Distributed File System) which stoes files used for processing but HDFS does not qualify as a relational database, it is just a storage model.

>There are components like Hive which can work on top of HDFS and which allows users to query the HDFS storage using SQL like queries using HiveQL but that is just SQL like queries and does not make HDFS or Apcahe Hadoop a database or relational database.

Thus, Hadoop is not a database even though it has a storage system HDFS. But can Hadoop replace the existing database? The answer is uncertain.

Firstly, schema is supported by relational database. Schema has its own advantages like it defines the contract of data format and it decouples the logical from physical. Moreover, with schema, there is no need to parse the data to get related fields. 

Secondly, column stores are good. Compared to the row stores, a column store has advantages like:

- `Read efficiency`: Usually only few columns will be selected for a query
- `Better compression`: Data under same column will have the same data type.
- `Vectorized processing`
>SIMD(Single Instruction Multiple Data)即单指令流多数据流，是一种采用一个控制器来控制多个处理器，同时对一组数据（又称“数据向量”）中的每一个分别执行相同的操作从而实现空间上的并行性的技术。简单来说就是一个指令能够同时处理多个数据。
- `Opportunities to operate directly on compressed data`



## HadoopDB
HadoopDB is an idea where co-locate a RDBMS on every slave node.


## Hive

- `Thrift`: To support schema on Hadoop
- `HDFS Block`: In each HDFS block, RCFile organizes the records with the basic unit of a row group. 
- `User Defined Functions (UDF)`: Provide index

## YARN - Yet Another Resource Negotiator
Yarn provides resource management and a central platform to deliver consistent operations, security and data governance tools across Hadoop clusters.

>By separating resource management functions from the programming model, YARN delegates many scheduling-related functions to per-job components.


## Dryad - Graph Operators
Dryad is a general-purpose distributed execution engine for coarse-grain data-parallel applications. 

## Spark
There are two problems with MapReduce:

- Programming language is Java
- Each MapReduce job will be written into disk, the shuffle is a bottleneck

Spark is a solution that extends the MapReduce.

## Presto
There are three roles in the Presto query execution:

- `Coordinator`: analyze and schedule
- `Discovery`: heartbeat and role managements
- `Worker`: execute

The query sent by the presto-cli is a HTTP POST request, and it will be sent to the coordinator. In side the coordinator, the query will be parsed to an Abstract Syntax Tree (AST), then analyzer will convert it to the logical query plan & distributed query plan, the optimizer will make it as optimized query plan, and sent to execution planner.

> 抽象语法树树，描述的最原始的用户需求。抽象语法树描述的信息，执行效率上不是最优，执行操作也过于复杂。需要把抽象语法树转化成执行计划。执行计划分成两类，一类是逻辑执行计划，一类是物理执行计划。逻辑执行计划，以树状结构来描述执行，每个节点是最简单的操作。物理执行计划，根据逻辑执行计划生成字节码，交由驱动执行。










<br/>
<br/>

## Reference
- [Is Hadoop a database](https://examples.javacodegeeks.com/enterprise-java/apache-hadoop/is-hadoop-a-database/)
- [深入浅出Hadoop YARN](https://zhuanlan.zhihu.com/p/55108455)
- [第八章 Hive的安装与数据导入导出](https://chu888chu888.gitbooks.io/hadoopstudy/content/Content/8/chapter8.html)
- [Hadoop的一些基本概念](https://chu888chu888.gitbooks.io/hadoopstudy/content/Content/3/chapter0301.html)
- [Presto实现原理和美团的使用实践](https://tech.meituan.com/2014/06/16/presto.html)
- [深入理解Presto(2) ：Presto查询执行过程](https://blog.csdn.net/sjtuyunlei/article/details/79382979)