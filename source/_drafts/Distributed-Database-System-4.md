---
title: Distributed Database System (4)
tags:
  - Distributed Database
  - Data Structure
date: 2020-11-22 12:40:10
---


This is revision note for NUS master study of CS5424 Distributed Databases.

## ACID

1. Atomicity: Either all or none of the actions in transactions happen (recovery manager)
2. Consistency: If each transaction is consistent and the DB starts consistent, it ends up consistent
3. Isolation: Execution of one transaction is isolated from other transactions (concurrency control manager)
4. Durability: If a transaction commits, its effects persist (recovery manager)

## Recovery Manager

- Commit(T): installs T’s updated pages into stable database
- Abort(T): restores all data that T updated to their prior values
- Restart(T): recovers database to a consistent state from system
failure
  - aborts all active Xacts at the time of system failure
  - installs updates of all committed Xacts that were not installed in the stable database before the failure

### Log-based Database Recovery

- Commit
  - Force-at-commit protocol
- Abort
  - Write-ahead logging (WAL) protocol

  在使用WAL的系统中，所有的修改在提交之前都要先写入log文件中

- Restart
  - Redo phase: scans log and keeps track of active Xacts
  - Undo phase: aborts all active Xacts


## Distributed Transactions

- Transaction coordinator (TC)
- Transaction manager (TM)

<br/>
{% asset_img TMTC.png TMTC %}
<br/>

## Failures in Distributed DBMS

- Site failures
  - Fail-stop model: A site is either working correctly (i.e.operational) or not working at all (i.e. failed)
  - Partial site failure: Some site(s) are operational & some site(s) are down
  - Total site failure: All sites are down
- Communication failures
  - Lost messages, network partitioning, etc.


## Two-Phase Commit (2PC) Protocol

- Voting/Preparation Phase: coordinator从participants那儿收集votes
- Decision Phase：coordinator会将global decision发送给所有的participants （只要有一个abort就全部abort）

## Dealing with Site Failures

### Recovery Protocols for 2PC

**Coordinator Site Failures**
- Fails in INITIAL state： 
  - 直接resume INITIAL状态
- Fails in WAIT state： 
  - 重新发送prepare消息
- Fails in COMMIT/ABORT state
  - 如果已经收到了所有的ACK消息，则不做任何事
  - 否则，将之前的Global-commit/Global-abort的决定再次发送

**Participant Site Failures**
- Fails in INITIAL state： 
  - 单方面abort
- Fails in READY state： 
  - 如果一个participant进入了READY状态，这就证明它之前已经发送了vote-commit
  - 重新发送vote-commit
- Fails in ABORT state：
  - 不做任何事
- Fails in COMMIT state：
  - 不做任何事

### Termination Protocols for 2PC

- Basic Termination Protocol
- Cooperative Termination Protocol

**Basic Termination Protocol**
- Cooordinator Timeouts
  - Timeout in WAIT state
    - 等待所有participant的vote的时候超时了，则写入log并且发送Global-abort
  - Timeout in COMMIT/ABORT state
    - 等待所有participant的ACK的时候超时了，则对没有回复的participant再次发送Global-commit/Global-abort
- Participant Timeouts
  - Timeout in INITIAL state
    - 等待prepare消息，可以单方面abort
  - Timeout in READY state
    - **必须等待coordinator的指令**

    > 2PC算法可以正确处理其他participant的failure，因为coordinator可以发送适当的指令。但是当coordinator宕机时，它必须等到coordinator发送消息才可以继续执行commit/abort，所以被阻塞了

**Cooperative Termination Protocol**

为了防止刚刚阻塞的情况发生，Cooperative Termination Protocol决定允许participant与其他participants沟通来获取commit/abort情况。

- Participant Timeouts
  - Timeout in READY state
    - 首先participant会有一个其他所有participants的地址列表
    - 发送decision-request消息给所有其他participants
      - 对于其他收到decision-request的participant Q 而言
      - 如果是INITIAL state，则回复"abort"
      - 如果是READY，则回复"Uncertain"
      - 如果是COMMIT/ABORT, 则回复决定
    - 收到任何决定（不管是Commit/abort）都执行
    - 如果也有收到"Uncertain"，则把决定传给那些不知道的participant

    > 这个protocol只能帮助减少阻塞，但不能完全解决。因为当所有人都在READY状态下，则大家还是必须等coordinator的指示。

## Three-Phase Commit (3PC) Protocol

- Voting/Preparation phase
  - Coordinator collects votes from participants
- Dissemination (传播) phase
  - Coordinator disseminates voting outcome to participants if there is no abort vote
- Decision phase
  - Coordinator sends global decision to participants

### Recovery Protocols for 3PC

**Coordinator Site Failures**
- Fails in INITIAL/PRECOMMIT state： 
  - 和participant询问
- Fails in COMMIT/ABORT state
  - 不做任何事

**Participant Site Failures**
- Fails in INITIAL state： 
  - 单方面abort
- Fails in READY/PRECOMMIT state： 
  - 和participant询问
- Fails in ABORT/COMMIT state：
  - 不做任何事

### Termination Protocols for 3PC

- Cooordinator Timeouts
  - Timeout in WAIT state
    - 等待所有participant的vote的时候超时了，则写入log并且发送Global-abort
  - Timeout in PRECOMMIT state
    - PRECOMMIT代表了所有的participant都vote for commit
    - 对没有回复的participant再次发送prepare-to-commit消息
    - 对其他有回复的participant发送Global-commit消息
  - Timeout in COMMIT state
    - Coordinator在等ACK
    - 因为已知能进入COMMIT状态代表了所有的participants都在PRECOMMIT/COMMIT，所以对其他没有回复的participant发送Global-commit消息
  - Timeout in ABORT state
    - Coordinator在等ACK
    - 因为已知能进入ABORT状态代表了所有的participants都在READY/ABORT，所以对其他没有回复的participant发送Global-abort消息

- Participant Timeouts
  - Timeout in INITIAL state
    - 可以单方面abort
  - Timeout in READY state
    - 执行New Coordinator Termination Protocol
  - Timeout in PRECOMMIT state
    - 执行New Coordinator Termination Protocol

**New Coordinator Termination Protocol**

选取新的coordinator。

- Participant Timeouts
  - Timeout in READY/PRECOMMIT state
    - 首先participant会有一个其他所有participants的地址列表
    - 选举出来的新的coordinator会发送State-request message消息给所有其他participants
      - 如果有些participant是COMMIT state，则对所有人发送"global-commit"
      - 否则，如果没有participant是PRECOMMIT state，则对所有人发送"global-abort"
      - 如果上面两种情况都没有，则对所有人发送Prepare-to-commit，收到Prepare-to-commit的话就对其他的participants发送"global-commit"