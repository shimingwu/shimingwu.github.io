---
title: Distributed Database System (5)
tags:
  - Distributed Database
  - Data Structure
date: 2020-11-22 16:44:23
---



This is revision note for NUS master study of CS5424 Distributed Databases.

## Transactions

- Single-partition transaction: a distributed transaction that access/update items from exactly one site
- Multi-partition transaction: a distributed transaction that access/update items from more than one site 

## Schedules

- Local schedule: transaction schedule at a local site
- Global Schedule: 
  - Let T = {T1, · · · , Tn} be a set of global transactions executed over m sites with local schedules {S1, · · · , Sm}
  - A schedule S is a global schedule for T and {S1, · · · , Sm} if each Si is a subsequence of S

## Concurrency Control Protocols

- Lock-based protocols
- Timestamp-based protocols
- Optimistic protocols
- Mutiversion protocols
- Hybrid protocols

# Distributed Lock-based Protocols

- Centralized 2PL (C2PL)
  - 1个site作为central site管理lock
  - Coordinating Site每次读 x 时，都和Central site说一声
- Distributed 2PL (D2PL)
  - 每个site上面都有一个lock manager，各自管理

## Two Phase Locking (2PL) Protocol

> 2PL schedules are conflict serializable

- Growing phase: before releasing 1st lock
- Shrinking phase: after releasing 1st lock

1. To read an object O, a Xact must hold a S-lock or X-lock on O
2. To write to an object O, a Xact must hold a X-lock on O
3. Once a Xact releases a lock, the Xact can’t request any more
locks

## Strict Two Phase Locking (S2PL)

> Strict 2PL schedules are conflict
serializable & recoverable

1. To read an object O, a Xact must hold a S-lock or X-lock on O
2. To write to an object O, a Xact must hold a X-lock on O
3. A Xact must hold on to locks until Xact commits or aborts

## Deadlock Detection

- Waits-for graph (WFG)
  - Deadlock is detected if WFG has a cycle

## Distributed Deadlock Detection

- Centralized approach
  - 每一个site都有一个Local Wait-For Graph (LWFG)
  - 阶段性地发给deadlock detector去检测。deadlock detector会将所有site的LWFG组合成Global Wait-For Graph (GWFG)并且检查是否有闭环。
- Distributed approaches

## Deadlock Prevention

- Each Xact is assigned a unique start timestamp
- Wait-die policy
  - 如果事务A晚于事务B开始，那么事务A等待另外一个事务释放对应资源的锁或者直接abort。
- Wound-wait policy
  - 如果事务A早于事务B开始，那么事务A等待，或者abort。这是一种抢占方案。


# Timestamp-based protocols

- Centralized Snapshot Isolation (CSI)
  - 一个site作为Centralized Coordinator (CC)，负责管理所有start和commit的时间戳（timestamp）的分发
  - TC会从CC上获取需要的timestamp，但是不管是对Write还是Read，lock都由TC各自管理
- Distributed Snapshot Isolation (DSI)

## Snapshot Isolation （SI）

- Write: 不同的transaction各自写各自的事务时，看谁的commit的时间更晚
- Read：只读commit最晚的

SI不一定是serializable。

## Multiversion Concurrency Control (MVCC)

在MVCC之前，主要采用"two-phase locking"，但"two-phase locking"过程中需要对读写操作进行加锁，这会带来显著的性能问题。因此，"two-phase locking"在读写高并发场景中已被逐步摒弃。

