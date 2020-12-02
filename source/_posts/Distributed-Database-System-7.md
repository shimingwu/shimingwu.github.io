---
title: Distributed Database System (7)
tags:
  - Distributed Database
  - Data Structure
date: 2020-11-26 16:21:47
---


This is revision note for NUS master study of CS5424 Distributed Databases.

## Raft Consensus Algorithm

### Server States:
- Follower
  - Passive but expects regular heartbeats from leader
- Candidate
  - Issues RequestVote RPCs to get elected as leader
- Leader
  - handles client interactions & issues AppendEntries RPCs
    - Replicate its log
    - Sends heartbeats to maintain leadership

## Term
Raft算法将时间分为一个个的任期（term），每一个term的开始都是Leader选举。在成功选举Leader之后，Leader会在整个term内管理整个集群。如果Leader选举失败，该term就会因为没有Leader而结束。

## Remote Procedure Calls (RPCs)
- RequestVote RPC
  - Candidate发送RequestVote RPC去请求成为leader
- AppendEntries RPC
  - Leader用AppendEntries RPC发送log和heartbeat

## Log完整度

X的log比Y的log更完整的情况只有：
1. X的log中的最后一个term比Y的log中的最后一个term大
2. term相同，但是X的最后一个index更大。
