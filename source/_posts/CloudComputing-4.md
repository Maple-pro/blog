---
title: '云计算：4. BigTable'
date: 2021-03-31 14:12:25
tags:
categories: [学习笔记, 云计算]
description: "记录 Bigtable 的基本数据模型：通过行关键字、列关键字与时间戳共同索引数据，将分布式结构化存储表示为多维映射表。当前内容聚焦数据模型。"
---

# 云计算：4. BigTable

## 1. 数据模型

**Bigtable数据的存储格式：**

Bigtable是一个分布式多维映射表，表中的数据通过一个行关键字（Row Key）、一个列关键字（Column Key）以及一个时间戳（Time Stamp）进行索引。

Bigtable的存储逻辑可以表示为：`(row:string, column:string, time:int64)→string`