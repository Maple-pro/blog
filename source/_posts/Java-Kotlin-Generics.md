---
title: Java & Kotlin Generics
tags: [Java, Kotlin]
categories: [学习笔记, Java]
date: 2022-06-30 17:42:59
mathjax:
description: "记录泛型学习中的 Java 通配符对照，通过 List<? extends E> 与 List<? super E> 比较读取、写入、生产者、消费者及协变、逆变的关系；Kotlin 泛型部分尚未展开。"
---

# Generics
## 1.1 Java

|                            | List<? extends E> | List<? super E> |
| -------------------------- | ----------------- | --------------- |
| read or write              | read              | write           |
| Producer or Consumer       | Producer          | Consumer        |
| covariant or contravariant | covariant         | contravariant   |

