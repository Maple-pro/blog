---
title: Java & Kotlin Generics
tags: [Java, Kotlin]
categories: [学习笔记, Java]
date: 2022-06-30 17:42:59
mathjax:
description: What's the difference of Generics between Java and Kotlin?
---

# Generics
## 1.1 Java

|                            | List<? extends E> | List<? super E> |
| -------------------------- | ----------------- | --------------- |
| read or write              | read              | write           |
| Producer or Consumer       | Producer          | Consumer        |
| covariant or contravariant | covariant         | contravariant   |

