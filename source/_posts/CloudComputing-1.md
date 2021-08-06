---
title: '云计算：1. 大数据与云计算'
date: 2021-03-15 16:44:51
tags: 
categories: [学习笔记, 云计算]
description: 介绍云计算和大数据
---

# 云计算：1. 大数据与云计算

## 1. IaaS vs PaaS vs SaaS

![preview](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/v2-1a82f8a4997b0ba53d801639d4c6706e_r.jpg)

- IaaS: Infrastructure as a Service
  - a virtual machine
- PaaS: Platform as a Service
- SaaS: Software as a Service

## 2. CAP Principle

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/20210303161406.png"  />

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/20210303161730.png"  />

- Consistency（一致性）
  - 一个写操作返回成功，那么之后的读请求都必须读到这个新数据；如果返回失败，那么所有读操作都不能读到这个数据。所有节点访问同一份最新的数据。
- Availability（可用性）
  - 对数据更新具备高可用性，请求能够及时处理，不会一直等待，即使出现节点失效。
- Partition Tolerance（分区容错性）
  - 能容忍网络分区，在网络断开的情况下，被分隔的节点仍能正常对外提供服务。

不可能都达成，最多只能满足其中两个。

理解CAP理论最简单的方式是想像两个副本处于分区两侧，即两个副本之间的网络断开，不能通信。

> - 如果允许其中一个副本更新，则会导致数据不一致，即丧失了C性质。
> - 如果为了保证一致性，将分区某一侧的副本设置为不可用，那么又丧失了A性质。
> - 除非两个副本可以互相通信，才能既保证C又保证A，这又会导致丧失P性质。

一般来说，使用网络通信的分布式系统，无法舍弃P性质，那么就只能在一致性C和可用性A上做一个艰难的选择。

## 3. BASE Theorem

- BA: Basically Available，基本可用
- S: Soft state，软状态
- E: Eventually consistent，最终一致

核心思想：如果无法做到强一致性，或者做到强一致性要付出很大的代价，那么应用可以根据自身业务特点，采用适当的方式来使系统达到最终一致性，只要对最终用户没有影响，或者影响是可接受的即可。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/20210303162448.png"  />

## 4. 奇数个节点（Zookeeper）

- 防止由脑裂造成的集群不可用
- 在容错能力相同的情况下，奇数台更节省资源