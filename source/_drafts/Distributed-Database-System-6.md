---
title: Distributed Database System (6)
tags:
  - Distributed Database
  - Data Structure
date: 2020-11-26 12:42:58
---


This is revision note for NUS master study of CS5424 Distributed Databases.

## One-Copy Database

- One-copy serializable (1SR)
- Mutual Consistency
  - Strong mutual consistency
  - Weak mutual consistency

## Replicas Update
- DBMS-level replication
  - Statement-based replication
    直接发statement
  - Write-ahead log (WAL) shipping 
- Application-level replication

## Replication Protocols

{% asset_img replication_protocols.png replication_protocols %}

### When are updates propagated to copies
- Eager (Synchronous) update:
  - Strong mutual consistency
  - Based on read-one-write-all (ROWA) protocols
- Lazy (Asynchronous) update:
  - Need to ensure that updates are applied in the **same order** to all replicas


### Where are updates allowed to occur
- Centralized technique:
  - Master copy updated first
- Distributed technique:
  - Any site can update first

### Protocols
- Eager Centralized Protocols
  - Eager Single-Master Protocol: 有一个single master，同时也是lock manager。
    - Read：由TC问一下master拿S-lock，从哪个site读都行
    - Write：由TC问一下master拿X-lock，master进行写操作，再保证其他site也进行写操作。
  - Eager Primary Copy Protocol: master有着primary copy。更新不同的数据，master不一样。其他和single-master一样。
- Eager Distributed Protocols
  - Read：先看看自己有没有数据备份，有的话，直接读，没有的话，则问别人
  - Write：先看看自己有没有数据备份，有的话，直接写，没有的话，则问别人
- Lazy Centralized Protocols
  - Lazy Single Master: There is a single master site containing master copies of all objects
  - Refresh transaction
  - 一个T被commit后，master告诉TC结束了，用同一个order对其他的site进行更新。更新使用refresh transaction的方式。
- Lazy Distributed Protocols

> In lazy protocol, the execution is not necessary 1SR

## Last-Writer-Wins Heuristic
为了避免写的冲突导致replicas的状态不一致，后发生的write会直接无视前一个write写的值直接执行覆盖。（blind write）

## Quorum Consensus (QC) Protocol
1. Tr (O) + Tw(O) > Wt(O)
2. 2 × Tw(O) > Wt(O)

1. The sum of read and write threshold must exceed the total weights of all replicas. This helps to ensure any next subsequent read operation will read an updated value performed by write.

2. 2 * Tw(O) > Wt(O) : Prevent two concurrent write operations.

## Handling Failures

- Failure of Slave Sites
  - Lazy replication
    - Synchronize unavailable replicas later when they become available
  - Eager replication
    - ROWA (Read-One/Write-All)
    - ROWAA (Read-One/Write-All Available)
      > Relaxation of ROWA protocol. Do not require updating of all replicas but only available replicas.
- Failure of Master Site
  - Elect a new master site
    - 选择想要的partition。如果多个partition且每个partition都有一个master，会产生数据不一致的问题。选择partition的算法：
      1. Simple algorithm
      2. Majority consensus algorithm
      3. Quorum consensus algorithm
    - 如果选择的partition没有可以工作的master，则选举新的master
      1. Consensus algorithm

### Simple Algorithm
选择有可用master的partition。

### Majority Consensus Algorithm
选择node数多的（大于一半数量）的partition。

### Quorum Consensus Algorithm
有权重。选择权重大的作为新的partition。

## CAP Theorem

### CAP
- Data Consistency
- System Availability
- Tolerance to Network Partitions

- Forfeit consistency (丧失一致性)
  - Resume execution on a selected partition
  - Data could become inconsistent if the selected partition requires a new master site
- Forfeit availability （丧失可用性）
  - Wait for network to recover before resuming execution