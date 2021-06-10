---
title: BigData(2)
tags:
  - Big Data
date: 2020-11-08 22:10:26
---


This is revision note for NUS master study of CS5425 Big Data Systems for Data Science.

## Challenges
In order to process the huge amount of data, parallelization is required. However, there are multiple challenge faced is the execution of the parallelization. For example, it is difficult to manage the communication between workers. One typical example can be the detection of a failed worker. It is hard to determine if a works is slow (takes longer to response) or it is failed. Moreover, there will have conflicts for resources access for different tasks. Even if there are multiple solutions like lock, conditional variables, etc, some other problems still exist.

It is a factor that there are many solutions provided, whilst these solutions do not fit as a big data solution. For example, programmers still need to handle the locks and take care of some performance optimizations. 

## MapReduce
With the challenges, MapReduce was proposed as **one of the solutions** (a programming framework) to solve the massive data problem. Google has implemented MapReduce in C++, Java and Python.

{% asset_img MapReduce.png illusion of MapReduce %}

Divide and conquer is the concept working behind. The tasks are distributed into different mappers and the tasks will be executed **in parallel**. However, the shuffle and sort becomes the bottleneck of the performance of the system because the moving of data is very costly. Last, the results will be sent to reducer for the aggregation, and the number of reducers can be another problem.

Besides the three steps mentioned:
- `Map`
- `Shuffle and sort`
- `Reduce`

Two more steps can be done: `combine` and `partition`. These two steps are serving as an optimization.

- `Combine`: The combine function will be done in **memory**. This is to reduce the amount of data to be transferred via network. For example, if we have multiple key-value pairs: {a:1}, {b:2}, {a:3}, {c:4}. There are total 4 pairs will be sent to the next step from current worker. A combine function can helps to combine the results to {a:4}, {b:2}, {c:4} if the method of combining is addition. The number of pairs to be sent will be decreased from 4 to 3. The combine function can happen at different levels after a mapper task is completed.

- `Partition`: The partition function influence how the shuffle and sort works. A partition function can divide/split the mapper results and based on key, value or the number of reducers to distribute the results. 

## Hadoop

As is known, MapReduce is a programming model and an associated implementation. Hadoop is an open-sourced version of MapReduce implementation in Java. 

There are two important parts in Hadoop:

- `HDFS`
- `MapReduce`

`HDFS` is a distributed file system and it is an open-sourced version of `GFS` (Google File System). GFS is the file system invested and used by Google to support a huge data processing and it has many advantages. As an open-sourced version (or clone) of the GFS, HDFS does not follow certain design decisions made by GFS. 

The architecture of HDFS is shown below.

{% asset_img HDFSArchitecture.png HDFS Architecture %}

> HDFS has a master/slave architecture. An HDFS cluster consists of a single NameNode, a master server that manages the file system namespace and regulates access to files by clients. In addition, there are a number of DataNodes, usually one per node in the cluster, which manage storage attached to the nodes that they run on. HDFS exposes a file system namespace and allows user data to be stored in files. Internally, a file is split into one or more blocks and these blocks are stored in a set of DataNodes. The NameNode executes file system namespace operations like opening, closing, and renaming files and directories. It also determines the mapping of blocks to DataNodes. The DataNodes are responsible for serving read and write requests from the file system’s clients. The DataNodes also perform block creation, deletion, and replication upon instruction from the NameNode.

Notice that HDFS is still a **master-slave architecture**.

## Basic Algorithm design with MapReduce

As MapReduce provides a high abstraction and many implementation details are hidden like we do not know where the mappers/reducers are running, when a particular mapper/reducers starts or finishes, there are only 4 places to implement an algorithm:

- `map`
- `reduce`
- `combiner`
- `partitioner`

Some concerns should be taken into considerations. For example, for partial results, we might not want to emit partial results immediately in certain cases, as this might result in more traffic over network and slow down the processing. Three things are important for the performance for an algorithm design for MapReduce:

- `Linear scalability`
- `Minimize I/O for both hard disk and network`
- `Memory`

### Linear Scalability
With the increase of the data size, the cost of communication will increase. As the **input to every reducer is guaranteed to be sorted** in MapReducer, there is a process to sort and transfer the results provided by mapper to the reducers, which is known as shuffle and it is a synchronization operation. The synchronous operation will make a significant impact on the overall performance. 

To avoid this, there are two main methods:

- `Local aggregation`
  
  A result can be aggregated during the map task. This is also known as `in-mapper combing` pattern.
- `Combiner`

  Use combiner to reduce the network traffic

For `Local aggregation`, it is usually faster than actual `combiner` because it does not need to spill to the disk. However, a higher memory is requested. Moreover, implementation complexity might increase.

<br/>
<br/>

## Reference
- [简单解释 MapReduce 算法](https://cloud.tencent.com/developer/article/1030329)  
- [HDFS Architecture Guide](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html)