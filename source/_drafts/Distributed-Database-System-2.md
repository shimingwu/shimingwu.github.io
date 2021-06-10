---
title: Distributed Database System (2)
tags:
  - Distributed Database
  - Data Structure
date: 2020-11-21 20:18:24
---


This is revision note for NUS master study of CS5424 Distributed Databases.

## Distributed Database Design 
  - `Data fragmentation / partitioning`: how to
partition data into smaller pieces
  - `Data allocation`: how to allocate data to various
sites
  - `Data replication`: what data to replicate at each
site

## Data Fragmentation
There are three strategies used:
  - Horizontal fragmentation (sharding)
  - Vertical fragmentation (Schema decomposition)
  - Hybrid fragmentation

### Desirable Properties of Fragmentation
  - Completeness: 
    - 在原R里面能找到的元素一定能在被碎片化后的R的set里面找到
    - `∀t ∈ R, ∃ Ri such that t ∈ Ri` (`∀: for all`, `∃: there exists exactly one`, for all ts belongs to set R, there exists one Ri such that t will belong to.)
  - Reconstruction：
    - 可以使用{R1, R2... Rn}还原R
    - `R = R1 ∪ · · · ∪ Rn`
  - Disjointness: 
    - Horizontal Decomposition: 如果一个元素在R1里面找到，它就不会出现在其他Rx里面，即R1与其他任何R的交集都是空集
    - Vertical Decomposition：这个属性只用于non-primary key
    - `∀ Ri ,Rj (i ！= j ⇒ Ri ∩ Rj = ∅)` 


## Horizontal fragmentation

### Techniques：


  - Range Partitioning
    - Partition R using predicates on some attributes
of R
    - R1 = σ A < 100 (R) (Select all elements in R that is smaller than 100)
    - R2 = σ A >= 100 (R) (Select all elements in R that is greater or equal than 100)
  - Hash Partitioning
    - Partition R into {R1, · · · ,Rn} based on hash function on some attribute of R (say R.A). 对某个attribute使用hash function来区分
    - Modulo method
      - Problems: 
        1. Uneven distribution
        2. Difficult to add in new server, need to rehash all existing entries
    - Consistent Hashing
      - Problems:
        1. Uneven distribution
        2. Heterogeneity of servers' performance
  - Primary Horizontal fragmentation
    - Fragmentation predicate Fi: Ri = σ Fi(R)
    - 根据已知需要执行的query来进行range partition
    - Primary Horizontal fragmentation是一个加强版的range partition
  - Derived Horizontal Fragmentation
    - 为了completeness, R.A ⊆ S.A
    - 为了disjointness, S.A must be a key


## Vertical fragmentation

### Attribute affinity measure

{% asset_img Attribute_Affinity_Measure.png Attribute_Affinity_Measure %}

Higher value means higher chance the 2 attributes will be queried together, therefore, we should group the attributes together.

## Hybrid fragmentation

Combinations of horizontal/vertical fragmentation.

## Complete Partitioning wrt Query

假设F = {R1, R2, R3.. Rn}。
假设Q是在R上面执行的query。
执行了Q后，得到的Rx包含了所有的Q所需要的tuples，以及没有包含任何Q所不需要的tuples，那么F是R对于Q的complete partitioning。

## Minterm Predicates
1. Let Q = {Q1, Q2... Qk} be a set of queries on relation R, where each Qi = σ pi (R)
2. Let P = {p1, p2... pk}
3. Let F = {R1, R2... ,Rm} be the minterm partitioning of R based on MTPred(P)
> Theorem: F is a complete partitioning wrt every
query in Q