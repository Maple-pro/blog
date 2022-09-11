---
title: '云计算：5. 分布式锁服务Chubby'
date: 2021-04-07 14:09:36
tags:
categories: [学习笔记, 云计算]
description: 介绍google的分布式锁服务---Chubby
---

> 这里我们讨论强一致性算法

# 1. Paxos算法

## 1.0 Glossary 

consensus 共识	quorum 法定人数

proposers, acceptors, learners

proposal, accept, majority, chosen, value

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210407142542380.png" alt="image-20210407142542380" style="zoom:80%;" />

## 1.1 算法的提出与证明

- 角色
  - proposers: 提出提案，提案信息包括提案编号和提议的value
  - acceptor: 收到提案后可以接受（accept）提案，若提案获得多数派（majority）的acceptors的接受，则称该提案被批准（chosen）
  - learners: 只能学习被批准的提案
- 条件
  - 决议（value）只有在被proposers提出后才能被批准（未经批准的决议称为提案（proposal））
  - 在一次Paxos算法的执行实例中，只批准（chosen）一个value
  - learners只能获得被批准（chosen）的value

## 1.2 算法的内容

**过程**：proposer提出一个提案前，首先要和足以形成多数派的acceptors进行通信，获得他们进行的最近一次接受（accept）的提案（prepare过程），之后根据回收的信息决定这次提案的value，形成提案开始投票。当获得多数acceptors接受（accept）后，提案获得批准（chosen），由acceptor将这个消息告知learner。

### 1.2.1 决议的提出和批准

两个阶段：

1. prepare阶段：
   1. proposer选择一个提案编号n并将prepare请求发送给acceptors中的一个多数派
   2. acceptors收到prepare消息后，如果提案的编号大于它已经回复的所有prepare消息(回复消息表示接受accept)，则acceptor将自己上次接受的提案回复给proposer，并承诺不再回复小于n的提案
2. 批准阶段：
   1. 当一个proposer收到了多数acceptors对prepare的回复后，就进入批准阶段。它要向回复prepare请求的acceptors发送accept请求，包括编号n和根据P2c决定的value（如果根据P2c没有已经接受的value，那么它可以自由决定value）
   2. 在不违背自己向其他proposer的承诺的前提下，acceptor收到accept请求后即批准这个请求

这个过程在任何时候中断都可以保证正确性。例如如果一个proposer发现已经有其他proposers提出了编号更高的提案，则有必要中断这个过程。因此为了优化，在上述prepare过程中，如果一个acceptor发现存在一个更高编号的提案，则需要通知proposer，提醒其中断这次提案。

### 1.2.2 例子

用实际的例子来更清晰地描述上述过程：

有A1, A2, A3, A4, A5 5位议员，就税率问题进行决议。议员A1决定降税率,因此它向所有人发出一个草案。这个草案的内容是：

```
现有的税率是什么?如果没有决定，我来决定一下. 提出时间：本届议会第3年3月15日;提案者：A1
```

在最简单的情况下，没有人与其竞争;信息能及时顺利地传达到其它议员处。

于是, A2-A5回应：

```
我已收到你的提案，等待最终批准
```

而A1在收到2份回复后就发布最终决议：

```
税率已定为10%,新的提案不得再讨论本问题。
```

这实际上退化为[二阶段提交](https://zh.wikipedia.org/wiki/二阶段提交)协议。

现在我们假设在A1提出提案的同时, A5决定将税率定为20%:

```
现有的税率是什么?如果没有决定，我来决定一下.时间：本届议会第3年3月16日;提案者：A5
```

草案要通过侍从送到其它议员的案头. A1的草案将由4位侍从送到A2-A5那里。现在，负责A2和A3的侍从将草案顺利送达，负责A4和A5的侍从则不上班. A5的草案则顺利的送至A4和A3手中。

现在, A1, A2, A3收到了A1的提案; A4, A3, A5收到了A5的提案。按照协议, A1, A2, A4, A5将接受他们收到的提案，侍从将拿着

```
我已收到你的提案，等待最终批准
```

的回复回到提案者那里。

而A3的行为将决定批准哪一个。

在讨论之前我们要明确一点，提案是全局有序的。在这个示例中，是说每个提案提出的日期都不一样。即第3年3月15日只有A1的提案；第3年3月16日只有A5的提案。不可能在某一天存在两个提案。

#### 情况一

假设A1的提案先送到A3处，而A5的侍从决定放假一段时间。于是A3接受并派出了侍从. A1等到了两位侍从，加上它自己已经构成一个多数派，于是税率10%将成为决议. A1派出侍从将决议送到所有议员处：

```
税率已定为10%,新的提案不得再讨论本问题。
```

A3在很久以后收到了来自A5的提案。由于税率问题已经讨论完毕，开始讨论某些议员在第3年3月17日提出的议案。因此这个3月16日提出的议案他不去理会。他自言自语地抱怨一句：

```
这都是老问题了，没有必要讨论了。
```

#### 情况二

依然假设A1的提案先送到A3处，但是这次A5的侍从不是放假了，只是中途耽搁了一会。这次, A3依然会将"接受"回复给A1.但是在决议成型之前它又收到了A5的提案。则：

1. 如果A5提案的提出时间比A1的提案更晚一些，这里确实满足这种情况，因为3月16日晚于3月15日。则A3回复：

```
我已收到您的提案，等待最终批准，但是您之前有人提出将税率定为10%,请明察。
```

于是, A1和A5都收到了足够的回复。这时关于税率问题就有两个提案在同时进行。但是A5知道之前有人提出税率为10%.于是A1和A5都会向全体议员广播：

```
 税率已定为10%,新的提案不得再讨论本问题。
```

共识到了保证。

2. 如果A5提案的提出时间比A1的提案更早一些。假设A5的提案是3月14日提出，则A3直接不理会。

```
 A1不久后就会广播税率定为10%.
```

### 1.2.3 决议的发布

一个显而易见的方法是当acceptors批准一个value时，将这个消息发送给所有learners。但是这个方法会导致消息量过大。

由于假设没有Byzantine failures，learners可以通过别的learners获取已经通过的决议。因此acceptors只需将批准的消息发送给指定的某一个learner，其他learners向它询问已经通过的决议。这个方法降低了消息量，但是指定learner失效将引起系统失效。

因此acceptors需要将accept消息发送给learners的一个子集，然后由这些learners去通知所有learners。

但是由于消息传递的不确定性，可能会没有任何learner获得了决议批准的消息。当learners需要了解决议通过情况时，可以让一个proposer重新进行一次提案。注意一个learner可能兼任proposer。

### 1.2.4 Progress的保证

根据上述过程当一个proposer发现存在编号更大的提案时将终止提案。这意味着提出一个编号更大的提案会终止之前的提案过程。如果两个proposer在这种情况下都转而提出一个编号更大的提案，就可能陷入活锁，违背了Progress的要求。一般活锁可以通过 **随机睡眠-重试** 的方法解决。这种情况下的解决方案是选举出一个leader，仅允许leader提出提案。但是由于消息传递的不确定性，可能有多个proposer自认为自己已经成为leader。Lamport在[The Part-Time Parliament](http://research.microsoft.com/users/lamport/pubs/lamport-paxos.pdf)一文中描述并解决了这个问题。

## 1.3 Multi-Paxos

Paxos的典型部署需要一组连续的被接受的值（value），作为应用到一个分布式状态机的一组命令。如果每个命令都通过一个Basic Paxos算法实例来达到一致，会产生大量开销。

如果Leader是相对稳定不变的，第1阶段就变得不必要。 这样，系统可以在接下来的Paxos算法实例中，跳过的第1阶段，直接使用同样的Leader。

为了实现这一目的，在同一个Leader执行每轮Paxos算法时，提案编号 I 每次递增一个值，并与每个值一起发送。Multi-Paxos在没有故障发生时，将消息延迟(从propose阶段到learn阶段)从4次延迟降低为2次延迟。

### 1.3.1 Multi-Paxos没有失败的情况

在下面的图中，只显示了基本Paxos协议的一个实例(或“执行”)和一个初始Leader(Proposer)。注意，Multi-Paxos使用几个Basic Paxos的实例。

```
Client   Proposer      Acceptor     Learner
   |         |          |  |  |       |  | --- First Request ---
   X-------->|          |  |  |       |  |  Request
   |         X--------->|->|->|       |  |  Prepare(N)
   |         |<---------X--X--X       |  |  Promise(N,I,{Va,Vb,Vc})
   |         X--------->|->|->|       |  |  Accept!(N,I,V)
   |         |<---------X--X--X------>|->|  Accepted(N,I,V)
   |<---------------------------------X--X  Response
   |         |          |  |  |       |  |
```

### 1.3.2 跳过阶段1的Multi-Paxos

在这种情况下，Basic Paxos的后续实例(由I+1表示)使用相同的Leader，因此，包含在Prepare和Promise的阶段1(Basic Paxos协议的这些后续实例)将被跳过。注意，这里要求Leader应该是稳定的，即它不应该崩溃或改变。

```
Client   Proposer       Acceptor     Learner
   |         |          |  |  |       |  |  --- Following Requests ---
   X-------->|          |  |  |       |  |  Request
   |         X--------->|->|->|       |  |  Accept!(N,I+1,W)
   |         |<---------X--X--X------>|->|  Accepted(N,I+1,W)
   |<---------------------------------X--X  Response
   |         |          |  |  |       |  |
```

## 1.4 Fast-Paxos



# 2. Raft

分布式选举算法

Redis哨兵机制



# 3. ZAB

原子广播 崩溃恢复

Paxos，Raft，2PC，3PC



弱一致性：Gossip协议



# 4. Zookeeper

curator

分布式锁：zookeeper redis



# Homework: 搭建Zookeeper集群

## 实现方法

本次使用了三台云服务器，分别为node-0001, node-0002, node-0003

- 在三台服务器上分别安装zookeeper环境。
- 在三台服务器的zoo.cfg文件末尾添加配置（由于三台服务器在同一网络下，因此直接使用内网IP）
  ```
  server.1=192.168.0.67:2888:3888
  server.2=192.168.0.23:2888:3888
  server.3=192.168.0.42:2888:3888
  ```
- 根据id和对应的IP地址分别配置myid
- 关闭防火墙
- 启动zookeeper

## 结果截图

集群中包含了三台云服务器，node-0001, node-0002, node-0003.

node-0001

![image-20210412204429224](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210412204429224.png)

node-0002

![image-20210412204534447](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210412204534447.png)

node-0003

![image-20210412204616901](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210412204616901.png)

可以发现有一个leader和两个follower。



# Reference

- [1] [维基百科：Paxos算法](https://zh.wikipedia.org/wiki/Paxos%E7%AE%97%E6%B3%95)
- [2] [硬货 | 浅谈 CAP 和 Paxos 共识算法](https://blog.csdn.net/u013256816/article/details/104809809)
- [3] [Raft协议原理详解](https://zhuanlan.zhihu.com/p/91288179)
- [4] [Raft动画过程演示](http://thesecretlivesofdata.com/raft/)
- [5] [Raft官网](https://raft.github.io/)
- [6] [Raft论文](https://raft.github.io/raft.pdf)
- [7] [分布式理论(七)—— 一致性协议之 ZAB](https://www.cnblogs.com/stateis0/p/9062133.html)