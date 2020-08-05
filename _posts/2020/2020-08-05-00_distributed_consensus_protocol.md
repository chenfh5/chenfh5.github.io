---
title: distributed consensus protocol
tags: algorithm
key: 113
article_header:
  type: cover
  image:
    src: https://user-images.githubusercontent.com/8369671/89384927-6ff64000-d731-11ea-9dc6-bf96b6f70848.png
---

# Overview
将自己对分布式一致性协议的理解, 简单记录一下,

先从问题<sup>8</sup>入手,

> 拜占庭将军问题(Byzantine Generals Problem), 拜占庭位于如今的土耳其的伊斯坦布尔, 是东罗马帝国的首都. 由于当时拜占庭罗马帝国国土辽阔, 为了达到防御目的, 每个军队都分隔很远, 将军与将军之间只能靠`信使`来传消息. <br>
在战争的时候, 拜占庭军队内所有将军和副官必须达成一致的共识, 决定是否有赢的机会才去攻打敌人的阵营. 但是, 在军队内有可能存有叛徒和敌军的间谍, 左右将军们的决定进而扰乱整体军队的秩序. 在进行共识时, 结果很可能并不能代表大多数良人的意见(分化, 田忌赛马). <br>
这时候, 在已知有成员谋反的情况下, 其余忠诚的将军在不受叛徒的影响下如何达成一致的协议, 拜占庭问题就此形成.

在计算机领域, 非黑客的情况下, 一般都会假设计算机之间不会互相发送恶意信息, 而更多的是宕机/网络延迟所造成的传信停止/滞后
> 所以将拜占庭将军问题根据常见的工作问题进行化简: 假设将军中没有叛军, 信使的信息可靠但有可能被暗杀的情况下, 将军们如何达成一致性决定?

# paxos
## 类型
- basic/single-decree paxos, 一次决策一个value, 仅在一个值上达成一致
- multi paxos, 连续决策多个value, 做到在一系列值上达成一致
    - 因为每个命令都通过一个Basic Paxos算法实例来达到一致, 会产生大量开销 
    - 类似kafka的batch send
    
```go
# Basic Paxos without failures
Client   Proposer      Acceptor     Learner
   |         |          |  |  |       |  |
   X-------->|          |  |  |       |  |  Request
   |         X--------->|->|->|       |  |  Prepare(N)
   |         |<---------X--X--X       |  |  Promise(N,{Va,Vb,Vc})
   |         X--------->|->|->|       |  |  Accept!(N,V)
   |         |<---------X--X--X------>|->|  Accepted(N,V)
   |<---------------------------------X--X  Response
   |         |          |  |  |       |  |

# Multi-Paxos without failures, 多出了提案编号I
Client   Proposer      Acceptor     Learner
   |         |          |  |  |       |  |
   X-------->|          |  |  |       |  |  Request
   |         X--------->|->|->|       |  |  Prepare(N)
   |         |<---------X--X--X       |  |  Promise(N,I,{Va,Vb,Vc})
   |         X--------->|->|->|       |  |  Accept!(N,I,V)
   |         |<---------X--X--X------>|->|  Accepted(N,I,V)
   |<---------------------------------X--X  Response
   |         |          |  |  |       |  |
```

## 角色
- client, 发出改值需求, 即需求方, 类似群众
- proposer, 随机选择一个节点, 将改值需求作为提案, 即提案者, 类似`基层`人大代表, 帮群众发声
- acceptor, 为提案投票, 即投票者, 类似全国人大代表, 负责审议表决
- learners, 记录最终提案并执行, 即记录员

## keypoint
-  提案号proposer number(n)越大, 案件越新, 越容易被接受, 一般是采用同步锁自增++的方式产生的, 最好能够保证n是全局唯一递增
- client request发送到系统, 随机抓取一个node作为proposer, 所以同一时刻可能有两个set请求
    - node1 -> set X=5
    - node2 -> set X=10
    - 但是quorum会拒绝n不大于自身offset的request, 因为这2个同时request可能会有一个会fail
    

## flow
![image](https://user-images.githubusercontent.com/8369671/89453148-5c7ac180-d791-11ea-95bd-7af2b2297bf6.png)
> proposal election and proposal workflow<sup>4</sup>

## paper<sup>1</sup>
```go
To ensure that only a single value is chosen, we can let a large
enough set consist of any majority of the agents. Because any two majorities
have at least one acceptor in common, this works if an acceptor can accept
at most one value.

Learning about proposals already accepted is easy enough; predicting 
future acceptances is hard.

Putting the actions of the proposer and acceptor together, we see that
the algorithm operates in the following two phases,
Phase 1,
    (a) A proposer selects a proposal number n and sends a prepare
        request with number n to a majority of acceptors.
    (b) If an acceptor receives a prepare request with number n greater
        than that of any prepare request to which it has already responded,
        then it responds to the request with a promise not to accept any more
        proposals numbered less than n and with the highest-numbered 
        proposal (if any) that it has accepted.
Phase 2, 
    (a) If the proposer receives a response to its prepare requests
        (numbered n) from a majority of acceptors, then it sends an accept
        request to each of those acceptors for a proposal numbered n with a
        value v, where v is the value of the highest-numbered proposal among
        the responses, or is any value if the responses reported no proposals.
    (b) If an acceptor receives an accept request for a proposal numbered
        n, it accepts the proposal unless it has already responded to a prepare
        request having a number greater than n.
```

# raft
是multi paxos的简化版本, 是对一系列连续问题达成一致的协议,
- 发送的请求是连续的, 即continuously append only
- 限制性选主. 必须有最新, 最全的日志节点才可以当选

## 角色
- follower, 响应candidate/leader的需求, 接受并持久化Leader同步过来的的日志 
    - election timeout
        > The election timeout is the amount of time a follower waits until becoming a candidate.<br>
          The election timeout is randomized to be between 150ms and 300ms.<br>
          After the election timeout the follower becomes a candidate and starts a new election term.
- candidate, 选举过程中的临时角色
- leader, 接收client的改值请求，并向follower同步请求日志，当日志同步到quorum节点后, 告诉follower可以提交日志(2PC)
    - heartbeat timeout
        > Once a candidate wins an election, it becomes leader.<br>
          It then sends heartbeat messages to all of the other servers to establish its authority and
          `prevent` new elections in a heartbeat timeout(50ms) interval.<br> 
          Once the leader crash and followers not receiving any heartbeat message during a random
          candidate timeout interval, then this election term will be terminated and one of the 
          followers would become a candidate.<br>

![image](https://user-images.githubusercontent.com/8369671/89632495-dc5d7480-d8d4-11ea-9694-1b410369c594.png)
> 角色状态转移关系<sup>9</sup>, credit mindwind

## keypoint
- 随机的election timeout, 导致同一时间点只能有一个leader(最早结束election timeout的就成为candidate, 并发起选自己为主的投票)
- 系统正常情况下每个任期有且仅有一个leader, 正常工作期间只有leader和followers
- client request发送到系统, 要么直接到leader或者到follower之后被redirect到leader
- 利用了once term one leader来tradeoff了multi-paxos的concurrency conflict和ZAB全局单leader的stability

## flow
![image](https://user-images.githubusercontent.com/8369671/89767985-8cc5b580-db2d-11ea-87df-56a0ab32799b.png)
> workflow

## paper<sup>9</sup>
```go
In Raft there are two timeout settings which control elections.

In normal operation there is exactly one leader and all of the other 
servers are followers.

Terms/任期 are numbered with consecutive integers increases monotonically 
over time. Each term begins with an election.

RequestVote RPCs are initiated by candidates during elections, and 
AppendEntries RPCs are initiated by leaders to replicate log entries 
and to provide a form of heartbeat.

In Raft, the leader handles inconsistencies by forcing the followers’ 
logs to duplicate its own. This means that conflicting entries in 
follower logs will be overwritten with entries from the leader’s log.

Logs are composed of entries, which are numbered sequentially.
Raft requires servers to apply entries in log index order.

Once a follower learns that a log entry is committed, it applies 
the entry to its local state machine.
```

# VR
Viewstamped Replication(VR),

VR | raft 
---- | ---- 
replicas| peers 
primary | leader 
backup | follower 
f+1 | quorum 
view | term
view number | logIndex
view change | re-election

## flow
![image](https://user-images.githubusercontent.com/8369671/89878993-dfb67000-dbf4-11ea-90bc-2997e9820eb2.png)
> here f=1, so quorum=1+1=2, so primary at least wait for 1 replica to response prepareOk

```go
VR ensures reliability and availability when no more
than a threshold of f replicas are faulty.

This implies that each step of the protocol must be processed by f + 1 replicas. 
These f + 1 together with the f that may not respond give us
the smallest group size of 2f + 1.

VR uses a primary replica to order client requests; the
other replicas are backups that simply accept the order
selected by the primary.
```

# ZAB
## similarity
- 类似raft
    ```go
    term -> epoch
    logIndex -> count
    ```
- 类似锁
    ```go
    分布式锁与领导选举/lock -> leadership
    排他锁eXclusive lock -> only one leader
    可重入锁reentrant lock -> 能再次被选举为leader并自己投自己

## keypoint
- follower/observer越多, 读性能越好, 但是如果保证f/o是最新的?
    - 写操作并不保证更新被所有的f/o立即确认, 因此通过部分f/o读取数据并不能保证读到最新的数据, 仅部分f/o及leader可读到最新数据
    - 如果一定要保证单一系统镜像, 可在读操作前调用sync()<sup>19</sup>
- log顺序性
- committed log sync/replicate
- leader一定拥有最大的zxid=epoch+count
- request过来
    - 正常情况, leader commited且不挂
    - commited了, 但是后续leader挂了, 那么max zxid的follower会继任
    - 未commit, 那么client超时, 然后重发, 此时zk cluster继任leader再receive req

## flow
![image](https://user-images.githubusercontent.com/8369671/89767992-90f1d300-db2d-11ea-9bc0-d58195283f46.png)
> workflow

## paper<sup>18</sup>
```go
Zookeeper has been built around a two-phase commit protocol that 
allows it to replicate all the transactions while keeping in mind 
all the design principles mentioned above. The leader node 
generates transactions and assigns sequel numbers to them 
upon receiving a client state change request. It then sends 
those transactions to all its follower nodes and waits for 
their acknowledgments.

When receiving ACKs from a quorum, commit calls are sent 
to the quorum for all the transactions. A follower checks 
the sequel number of the issued transaction and only 
commits it if it doesn’t have any outstanding 
transactions in the queue.

A node can only be a leader/master node if it has the quorum 
number of nodes as followers.



An outstanding transaction is one that has been proposed 
but not yet delivered

The original Paxos protocol does not enable multiple outstanding transactions.
Paxos does not require FIFO channels for communication, so it tolerates message loss
and reordering. If two outstanding transactions have an order dependency, then Paxos
cannot have multiple outstanding transactions because FIFO order is not guaranteed.
This problem could be solved by batching multiple transactions into a single proposal
and allowing at most one proposal at a time, but this has performance drawbacks.
```

# ZEN
zen discovery主要用于<sup>21</sup>,
- discovering nodes/peers
    ```go
    At startup, or when electing a new master, Elasticsearch tries to connect to each 
    `seed node` in its list, and holds a gossip-like conversation with them to find 
    other nodes and to build a complete picture of the cluster.
    
    PeerFinder.startProbe().peersByAddress.computeIfAbsent(transportAddress, this::createConnectingPeer);
    
    - peers通过seed nodes这个中间节点来发现彼此
    - peers彼此交换master-eligible nodes
    ```
- electing a master(quorum)
    ```java
     /** master nodes go before other nodes, with a secondary sort by id **/
    private static int compareNodes(DiscoveryNode o1, DiscoveryNode o2) {
        if (o1.isMasterNode() && !o2.isMasterNode()) {
            return -1;
        }
        if (!o1.isMasterNode() && o2.isMasterNode()) { // 固定配置, master or data or ingest
            return 1;
        }
        return o1.getId().compareTo(o2.getId());
    }
     
    The responsibility of the master node is to maintain the global cluster state 
    and reassign shards when nodes join or leave the cluster. Each time the 
    cluster state is changed, the new state is published to all nodes in the 
    cluster as described above.
    ```
- forming a cluster
- publishing cluster state

## flow
![image](https://user-images.githubusercontent.com/8369671/89873333-db865480-dbec-11ea-960c-2bcf18e90b44.png)
> discovering

# gossip


# Reference
0. [Paxos Made Simple](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)
0. [Paxos wiki](https://en.wikipedia.org/wiki/Paxos_(computer_science)#History)
0. [漫话分布式系统共识协议: Paxos篇](https://zhuanlan.zhihu.com/p/35737689)
0. [图解分布式一致性协议Paxos](http://codemacro.com/2014/10/15/explain-poxos/)
0. [Paxos算法](https://www.cnblogs.com/liuyi6/p/10703480.html)
0. [Message Passing vs Shared Memory Process communication Models](https://www.tutorialspoint.com/message-passing-vs-shared-memory-process-communication-models)
0. [paxos github code](https://github.com/allenfromu/Single-Decree-Paxos)
0. [Raft理论基础](https://zhuanlan.zhihu.com/p/85680181)
0. [In Search of an Understandable Consensus Algorithm](https://raft.github.io/raft.pdf)
0. [raft live](http://thesecretlivesofdata.com/raft/)
0. [Raft算法详解](https://zhuanlan.zhihu.com/p/32052223)
0. [理解Copy On Write技术](http://wsfdl.com/algorithm/2016/09/29/Copy_on_write.html)
0. [raft github code](https://github.com/ktoso/akka-raft)
0. [Viewstamped Replication Revisited](http://pmg.csail.mit.edu/papers/vr-revisited.pdf)
0. [Want to learn how Viewstamped Replication works? Read this summary](https://www.freecodecamp.org/news/viewstamped-replication-revisited-a-summary-144ac94bd16f/)
0. [vr github code](https://github.com/hgadre/viewstamped-replication)
0. [ZooKeeper’s atomic broadcast protocol: Theory and practice](http://www.tcs.hut.fi/Studies/T-79.5001/reports/2012-deSouzaMedeiros.pdf)
0. [Zookeeper Atomic Broadcast Protocol (ZAB) and implementation of Zookeeper](https://www.cloudkarafka.com/blog/2018-07-04-cloudkarafka-zab.html)
0. [ZooKeeper Programmer's Guide](https://zookeeper.apache.org/doc/r3.2.2/zookeeperProgrammers.html#ch_zkGuarantees)
0. [zab github code](https://github.com/zk1931/jzab)
0. [es discovery and cluster formation](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/modules-discovery.html)
0. [zen github code](https://github.com/elastic/elasticsearch/tree/v7.8.1/server/src/main/java/org/elasticsearch/discovery)
