---
title: Distributed Database System (1)
tags:
  - Distributed Database
  - Data Structure
date: 2020-11-21 18:52:59
---


This is revision note for NUS master study of CS5424 Distributed Databases.

## Modern Distributed DBMS

### History
- Relational DBMS:
  - Initially targeted at business processing
applications 
  - OLTP = On-Line Transaction Processing
  - Characteristics: small update ACID transactions
- Early distributed DBMS:
  - Targeted to support the organizational structure
of distributed enterprises
- Parallel DBMS:
  - Targeted at decision support systems (DSSs)
  - OLAP = On-line Analytical Processing
  - Characteristics: Complex read-mostly queries on large data

### NewSQL Database Systems
- Targeted at OLTP workloads
- Features
  - Relational data model
  - SQL query language
  - ACID transactions
  - Runs on distributed cluster of shared-nothing nodes

> NewSQL Database Systems: Relational DB can be scaled. It is designed in a distributed way to provide rational DB.

## Review of Relational Algebra

- `σA>5`
  
  SELECT * FROM R WHERE A > 5
- `πX,Y,Z (R)`

  SELECT DISTINCT X, Y, Z FROM R

- `R ⊳⊲A S`

  SELECT * FROM R JOIN S ON R.A = S.A

- `R ⋉A S`

  SELECT * FROM R WHERE EXISTS (SELECT * FROM S WHERE R.A = S.A)

## Review of ACID Transactions
- Atomicity: Transaction is either executed completely or not at all
- Consistency: Transaction preserves database consistency
- Isolation: Execution of a Transaction is isolated from other Transaction
- Durability: If a Transaction commits, its effects persist


## Serial Schedules:
   - 所有事务排队，没有并发

## Serializable Schedule：
   - 有并发，但存在一个完全等价的serial schedule
   - 往往指代conflict serializable，即没有以下：
     - Read-write conflict
     - Write-read conflict
     - Write-write conflict
