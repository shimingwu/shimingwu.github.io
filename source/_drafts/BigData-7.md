---
title: BigData(7)
tags:
  - Big Data
date: 2020-11-10 21:30:24
---



This is revision note for NUS master study of CS5425 Big Data Systems for Data Science.

## Link Analysis Algorithms

There are three algorithms involved to compute the **importance** of nodes in the graph.

- `Page Rank`
- `Topic-Specific (Personalized) Page Rank`
- `Web Spam Detection Algorithms`

## Page Rank
In page rank, the higher the score, the more important a node is. There is one important property of this algorithm, **a page is important if it is pointed by other important pages.**

{% asset_img flow_equation.png flow_equation %}

A flow equation is used to describe the rank.
```
For node a, we can see that there are 2 outgoing links, each of the link will have a/2.
Similarly, node y has two outgoings, so each link is y/2.
For node m, only one outgoing, thus, link has m.

To get the rank of y, sum up all incoming links:
y/2 + a/2

To get the rank of a, sum up all incoming links:
y/2 + m

To get the rank of m, sum up all incoming links:
a/2
```
Thus, the flow equation is:
```
ry = ry/2 + ra/2
ra = ry/2 + rm
rm = ra/2
```
As there are three unknowns, three equations without constants, we add one more constraint to force the uniqueness:
```
ry + ra + rm = 1
```
With this, we are able to calculate the page rank. However, the calculation will be very heavy when there is a large number of nodes.

## Power iteration
There are two parts involved in `power iteration`:

- `Stochastic adjacency matrix M`
- `Eigenvector`

From above, we can build a stochastic adjacency matrix M:

{% asset_img flow_equation_M.png flow_equation_M %}

Apply the power iteration:

{% asset_img power_iteration.png power_iteration %}

## Random Teleports
1. **Dead Ends**. The result might not converge:
Node `A` points to Node `B` and Node B points to Node `A`, `A` <---> `B`

2. **Spider traps**. Importance information leaks out:
Node `A` points to Node `B`, `A` ---> `B`

To solve these, `Teleports` is introduced by Google. The solution is that in each 'flow', it has certain opportunity to jump to other nodes (not the linked one).

{% asset_img illustration_random_teleport.png illustration_random_teleport %}
{% asset_img illustration_random_teleport_calculation.png illustration_random_teleport_calculation %}

## Topic-Specific PageRank

The evaluation is not just conducted for the popularity but on the topic as well. To achieve this, the probability assigned for teleport is not consistent for all nodes. Instead, **when a walker teleports, only a page in a set S which only includes pages are relevant to the topic will be picked.**

## SimRank

>Effectively, SimRank is a measure that says "two objects are considered to be similar if they are referenced by similar objects."

## TrustRank

- `Link Farms`: `Inaccessible pages`, `Accessible pages` and `Owned pages`

## HITS
- `Hubs` and `Authorities`

**Hubs**
Quality as an expert. Total sum of votes for authorities pointed to.

**Authority**
Quality as a content. Total sum of votes coming from experts.

> Authorities are pages containing useful information like Newspaper home pages, Course home pages and Home pages of auto manufacturers

> Hubs are pages that link to authorities like List of newspapers, Course bulletin and List of US auto manufacturers

{% asset_img HITS_flow.png HITS_flow %}

{% asset_img HITS.png HITS %}

## Difference between PageRank and HITS

- In the PageRank model, the value of the link depends on the **links into u**
- In the HITS model, it depends on the value of the other **links out of u**

## Pregel
Pregel is serving as a computational model. There are four basic elements in Pregel model:

- `Vertex`: In Pregel, each node(vertex) has an unique ID
- `Edge`: In Pregel, each edge can be assigned with a property
- `Message`: Message is the core design in Pregel. Message is used to indicate the status of a vertex.
- `Superstep`: One superstep is an iteration of computation happens in the Pregel.

In Pregel, for each vertex at each superstep, it will receive a set of messages from previous superstep. The vertex will process all the messages then emit these messages to other vertex.


<br/>
<br/>

## Reference
- [Pregel总结](https://cloud.tencent.com/developer/news/148367)
- [图解图算法 Pregel: 模型简介与实战案例](https://io-meter.com/2018/03/23/pregel-in-graphs/)
- [Pregel（图计算）技术原理](https://cshihong.github.io/2018/05/30/Pregel%EF%BC%88%E5%9B%BE%E8%AE%A1%E7%AE%97%EF%BC%89%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86/)

