---
title: BigData(6)
tags:
  - Big Data
date: 2020-11-10 13:24:56
---


This is revision note for NUS master study of CS5425 Big Data Systems for Data Science.

## Spark
Spark generalizes the MapReduce to provide the following features:

- `High performance`
- `Better programmability`
- `A unified engine`

## RDD (Resilient Distributed Datasets)
RDD is the core abstraction. 
> Collections of objects spread across a cluster, stored in RAM or on Disk. Built through parallel transformations and automatically rebuilt on failure

There are two types of RDD:

- `Parallelized collections`: take an existing single-node collection and parallel it
- `Hadoop datasets`: files on HDFS or other compatible storage

The operations on RDDs are not computed immediately. Instead, only certain actions will trigger the computation, for example, `count`, `saveAsTextFile`.

RDDs track lineage information to reconstruct lost partitions.

## DataFrame

Spark SQL is a Spark module for structured data processing. There are several ways to interact with Spark SQL including SQL and the Dataset API and with the extra information to perform extra optimizations, and a DataFrame is a Dataset organized into named columns. The image below shows the general idea.

{% asset_img SparkSQL.png SparkSQL %}

DataFrames hold a row with schema information and provides a relational operations regarding the schema.

## Directions for Spark

- `Multi-core scalability`
- `Continuous (streaming) applications`

## Challenges faced for numerical computation over big data

- `Correctness`
- `Performance`: Cache RDDs to avoid I/O and avoid unnecessary computation
- `Trade-off between accuracy and performance`

## Spark libraries
There are many libraries in Spark to support common algorithms, like GraphX, MLlib, etc.

- `GraphX`: A general graph processing library. It builds the graph using RDDs of nodes and edges.
- `Spark streaming`: It will split the live stream into different chunks and each chunk of data will be treated as RDDs and the processed results will be returned in batches.
- `Spark SQL`: Enables to save and query the structured data in Spark.
```
{
  "text": "Example",
  "user": {
    "name": "Mike",
    "id:1
  }
}

To query from Json, we can have:
c.sql("select user.name from tweets")
```
- `MLlib`: Support different machine learning algorithms








<br/>
<br/>

## Reference
- [Spark SQL, DataFrame 和 Dataset 编程指南](https://spark-reference-doc-cn.readthedocs.io/zh_CN/latest/programming-guide/sql-guide.html)
- [Spark SQL, DataFrames and Datasets Guide](https://spark.apache.org/docs/latest/sql-programming-guide.html)
- [spark2.0原理分析--RDD血缘（RDD Lineage）](https://blog.csdn.net/zg_hover/article/details/73159212)