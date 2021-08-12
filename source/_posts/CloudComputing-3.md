---
title: '云计算：3. 容器'
date: 2021-03-17 14:12:43
tags: [Docker]
categories: [学习笔记, 云计算]
description: 介绍云计算和大数据
---

# 1. Docker镜像

- 容器镜像打包了应用及其依赖（包含完整操作系统的所有文件和目录）
- 容器镜像包含了应用运行所需要的所有依赖。
- 实现了本地环境与云端环境的高一致性
- 通过6个Namespace和Cgroup为每个应用创建隔离的运行环境

![image-20210317142915755](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210317142915755.png)

# 2. Container VS VM (Virtual Machine)

![image-20210317143240171](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210317143240171.png)

- Virtual Machine
  - Hypervisor：例如Win10等操作系统
  - Guest Operating System：消耗资源很大
  - 优点：隔离性很好
  - 缺点：CPU、硬盘、内存占用高，启动速度慢
- Container
  - Host Operating System: Linux操作系统
  - Docker: Docker Engine
  - App: Docker image
  - 优点：占用资源小，响应速度快
  - 缺点：隔离性没有VM好（一定是缺点吗？）

![image-20210317144952233](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210317144952233.png)



# 3. 容器编排引擎

Kubernetes, MESOS, Docker Swarm



# 4. 容器网络



# 5. 容器存储



# 6. 容器底层实现技术



