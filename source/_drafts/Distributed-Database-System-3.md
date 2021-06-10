---
title: Distributed Database System (3)
tags:
  - Distributed Database
  - Data Structure
date: 2020-11-21 21:09:22
---


This is revision note for NUS master study of CS5424 Distributed Databases.

## LSM Storage
LSM storage for a relation R(K, V) consists of:
- A main-memory structure MemTable
- A set of disk-based structures SSTables
- A commit log file

### MemTable (Memory Table)
  - Contains the most recent updates organized in main-memory
  - MemTable is updated in-place
    - Deleted records are not removed but marked with tombstones (denoted by ⊥)
  - When size of MemTable reaches a certain threshold, the records in MemTable are sorted and flushed to disk as a new SSTable

### SSTable (Sorted String Table)
  - SSTables are immutable structures
  - SSTable records are **sorted** by relation’s key K
  - Each SSTable is associated with a range of key values & a timestamp

### Commit Log File
  - A commit log file is used to ensure durability
  - Each new update is appended to commit log & updated to MemTable

## Compaction of SSTables
Compaction helps to improve:
- Read performance by defragmenting table records
- Improves space utilization by eliminating tombstones & stale values

### Compaction Strategies
- Size-tiered Compaction Strategy (STCS)
- Leveled Compaction Strategy (LCS)

## Local vs Global Indexing
Advantage of using local index: Update will be easier with local index. The index is associated with data stored in the same site.

Advantage of using global index, search will be easier. We only need to retrieve all the entries from one site.

## Secondary Index
- B+-tree
  - Updates are performed in-place 
  - Posting list for each key is stored together
  - Example: MongoDB
- LSM Storage 
  - Updates are performed with append-only updates
  - Posting list for each key could be fragmented across multiple SSTables
  - Example: Cassandra