---
title: BigData(3)
tags:
  - Big Data
date: 2020-11-09 14:24:09
---



This is revision note for NUS master study of CS5425 Big Data Systems for Data Science.

## Relational Database
Usually, Hadoop/MapReduce is used to support the OLAP.

### Relational Algebra

Considering the table `Employee`. All examples will use the table.

**Projection Π**

<table>
<tr><th> Employee </th><th> Π NAME, AGE (Employee) </th></tr>
<tr><td>

| ID    | NAME | AGE |
| :---- | :----:  | :---- |
| 1 | Mike | 12 |
| 2 | Jane | 22 |
| 3 | Vincent | 45 |

</td><td>

| NAME | AGE |
| :----:  | :---- |
| Mike | 12 |
| Jane | 22 |
| Vincent | 45 |

</td></tr> </table>

To perform projection, only mapper is used. The bottleneck is the streaming read speed of HDFS.

**Selection σ**

<table>
<tr><th> Employee </th><th> σ ID = 1 (Employee) </th></tr>
<tr><td>

| ID    | NAME | AGE |
| :---- | :----:  | :---- |
| 1 | Mike | 12 |
| 2 | Jane | 22 |
| 3 | Vincent | 45 |

</td><td>

| ID    | NAME | AGE |
| :---- | :----:  | :---- |
| 1 | Mike | 12 |

</td></tr> </table>

To perform projection, only mapper is used. The bottleneck is the streaming read speed of HDFS. Selection is very similar to Projection in general.

**Group By (Aggregation)**

This can be done via mapper + reducer. In addition, combiner can be used to optimize the performance. 

**Order By**

There are two sort problems here: Sort by **Key** and Sort by **Value**.

In terms of sort by key, MapReduce (Hadoop) guarantees the order of keys are sorted before sending to the reducers. If a tuple only has a single column, for example, {'Apple', 1}, then the sort is very simple.

However, what if there are multiple columns exist for a tuple, for example, {'Apple', '2020-10-2', 1}. If we want to sort by the fruit name, followed by the sort of date, there are 2 main approaches:

- `Sort inside Reducer (BAD)`
- `Value-to-Key (BETTER)`

> For the first approach, it involves having the reducer buffer all of the values for a given key and do an in-reducer sort. Since the reducer will be receiving all values for a given key, this approach could possibly cause the reducer to run out of memory. 

<table>
<tr>
  <th> Map </th>
  <th> Shuffle and Sort (by the key) </th>
  <th> Reduce </th>
</tr>
<tr>

<td>

| Key    | Value 1 | Value 2 |
| :---- | :----:  | :---- |
| Apple | '2020-10-2' | 10 |
| Banana | '2020-10-5' | 20 |
| Orange | '2020-10-2' | 10 |
| Apple | '2020-10-5' | 20 |
| Apple | '2020-10-2' | 10 |
| Banana | '2020-10-5' | 20 |
| Orange | '2020-10-2' | 10 |
| Apple | '2020-10-5' | 20 |
</td>
<td>

| Key    | Values under the key | 
| :---- | :----:  | 
| Apple | ('2020-10-2', 10)<br/> **('2020-10-5', 20)**<br/> **('2020-10-2', 10)**<br/> ('2020-10-5', 20) | 
| Banana | ('2020-10-5', 20)<br/> ('2020-10-5', 20) | 
| Orange | ('2020-10-2', 10)<br/> ('2020-10-2', 10) | 

</td>
<td>

| Key    | Values under the key | 
| :---- | :----:  | 
| Apple | ('2020-10-2', 10)<br/>  **('2020-10-2', 10)**<br/> **('2020-10-5', 20)**<br/>('2020-10-5', 20) | 
| Banana | ('2020-10-5', 20)<br/> ('2020-10-5', 20) | 
| Orange | ('2020-10-2', 10)<br/> ('2020-10-2', 10) | 
</td>
</tr> 

</table>

>The second approach involves creating a composite key by adding a part of, or the entire value to the natural key to achieve your sorting objectives. 

<table>
<tr>
  <th> Map </th>
  <th> Shuffle/Partition </th>
  <th> Sort </th>
</tr>
<tr>

<td>

| Key    | Value 1 | Value 2 |
| :---- | :----:  | :---- |
| Apple | '2020-10-2' | 10 |
| Banana | '2020-10-5' | 20 |
| Orange | '2020-10-2' | 10 |
| Apple | '2020-10-5' | 20 |
| Apple | '2020-10-2' | 10 |
| Banana | '2020-10-5' | 20 |
| Orange | '2020-10-2' | 10 |
| Apple | '2020-10-5' | 20 |
</td>
<td>

| Key    | Value 2 |
| :----:  | :---- |
| Apple, '2020-10-2' | 10 |
| Apple,**'2020-10-5'** | 20 |
| Apple, **'2020-10-2'** | 10 |
| Apple, '2020-10-5' | 20 |
| Banana, '2020-10-5' | 20 |
| Banana, '2020-10-5' | 20 |
| Orange, '2020-10-2' | 10 |
| Orange, '2020-10-2' | 10 |


</td>
<td>

| Key | Value 2 |
| :----:  | :---- |
| Apple, '2020-10-2' | 10 |
| Apple, **'2020-10-2'** | 20 |
| Apple, **'2020-10-5'** | 10 |
| Apple, '2020-10-5' | 20 |
| Banana, '2020-10-5' | 20 |
| Banana, '2020-10-5' | 20 |
| Orange, '2020-10-2' | 10 |
| Orange, '2020-10-2' | 10 |
</td>
</tr> 

</table>

**NATURAL JOIN (⋈)**

There are three approaches:

- `Reduce-side Join`
- `Map-side Join`
- `In memory Join`

**Reduce-side Join**

When a join is performed by the reducer, it is called as reduce-side join. For this method, the main thing done on the map stage is put a `tag` to the files. For example:

{% asset_img reduce_side_join_example.png Example %}

Another illustration is taken from Guru99.

{% asset_img reduce_side_join_from_guru99.png Example %}

In general, this method will work for 1-to-1 join. However, for 1-to-many or many-to-many, it might have a potential problem with memory. To solve the problem, there are two keys:

- `Save the data into memory, if the memory is not enough, write to disk`
- `Value-to-Key conversion`

**Map-side Join**

No reducer is required. 
> Suppose we have two datasets that are both sorted by the join key. We can perform a join by scanning through both datasets simultaneously. This is known as a merge join in the database community. We can parallelize this by partitioning and sorting both datasets in the same way. 

**In memory Join**

Load one dataset into memory, stream over other dataset.

In general, `in-memory join` > `map-side join` > `reduce-side join`.

Best choice: when the data is smaller than main memory, prefers in-memory join. If in-memory join is not possible, yet, the file is partitioned according to the key, uses map-side join. Otherwise, use reduce-side join. 




<br/>
<br/>

## Reference
- [Relational Algebra in DBMS: Operations with Examples](https://www.guru99.com/relational-algebra-dbms.html)  
- [Hadoop MapReduce Join & Counter with Example](https://www.guru99.com/introduction-to-counters-joins-in-map-reduce.html)  
- [MapReduce：实现join的几种方法](https://blog.csdn.net/sofuzi/article/details/81265402)  
- [深入理解 Reduce-side Join](http://shzhangji.com/cnblogs/2015/01/13/understand-reduce-side-join/)


