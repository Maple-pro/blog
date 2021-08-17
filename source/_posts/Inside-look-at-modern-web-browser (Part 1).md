---
title: 深入理解现代 Web 浏览器 (Part 1)
date: 2021-08-17 22:04:04
tags: [浏览器]
categories: [前端]
mathjax: false
description: "介绍 Chrome 浏览器的架构以及 Servicification 和 Site Isolation 特性。<br/>翻译自: https://developers.google.com/web/updates/2018/09/inside-browser-part1"
---

# Part 1. CPU, GPU, 内存和多线程架构

> Reference: https://developers.google.com/web/updates/2018/09/inside-browser-part1

## 计算机的核心是 CPU 和 GPU

为了理解浏览器运行的环境，我们需要理解一些计算机相关的知识以及它们是做什么的。

### CPU

首先是中央处理器（CPU）。CPU 可以被认为是计算机的大脑。一个 CPU 的内核，可以理解为一个员工，他可以一个接一个地处理输入进来的不同任务。它可以处理从数学到艺术的所有事情，同时也知道如何回复客户的电话。在过去，很多 CPU 都是单个芯片。一个内核就像是在同一个芯片的另一个 CPU。在现代的硬件中，通常是多核，使得手机和电脑有了更强的计算能力。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210817224636528.png" alt="image-20210817224636528" style="zoom:80%;" />

### GPU

图形处理器（GPU）是计算机中的另外一部分。不像 CPU，GPU 更擅长处理简单的事物，但能同时跨多个内核。就像它的名称描述的那样，它最开始是被用来做图像处理的。这也就是为什么『使用 GPU』和『GPU 支持』的图形内容往往与快速渲染和流畅交互相关联。近几年，伴随着 GPU 加速计算，越来越多的计算有了可能性。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210817224701308.png" alt="image-20210817224701308" style="zoom:80%;" />

当你在电脑或手机上打开一个应用时，CPU 和 GPU 驱动了程序的运行。通常来说，应用运行在 CPU 和 GPU 上需要使用到操作系统提供的机制。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210817224909891.png" alt="image-20210817224909891" style="zoom: 80%;" />

## 程序在进程和线程上运行

在深入浏览器架构之前，另一个需要掌握的概念是『进程』和『线程』。进程可以被看作是一个应用的执行程序。线程是进程的一部分，并且执行进程中的任意一部分程序。

当我们启动应用时，进程会被创建。程序可能会创建多个线程去帮助它工作，但这是可选的。操作系统给进程一块内存空间用于执行，并且程序的所有状态信息都存放在这个私有的内存空间中。当程序关闭时，进程也会被杀死，同时分配的内存空间也会被释放。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210817230837103.png" alt="image-20210817230837103" style="zoom:80%;" />

一个进程可以请求操作系统启动另外一个进程来运行不同的任务。当上述事件发生时，另一部分的内存会被分配给新的进程。如果两个进程间需要通信，他们可以通过『进程间通信』（IPC）的方式来通信。许多应用都被设计成这样，因此当一个工作进程无响应时，它可以被另外一个进程唤醒，而不需要重启应用。

## 浏览器架构

那么 Web 浏览器时如何通过进程和线程来构建的呢？它可以是一个进程中有多个线程，或者是多个不同的进程通过 IPC 来通信。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210817231528432.png" alt="image-20210817231528432" style="zoom:80%;" />

这里需要注意的是，这些不同的架构是浏览器的实现细节。这里没有一个构建 Web 浏览器的标准。一个浏览器的实现方法可能和另外一个浏览器完全不同。

在这篇博客中，我将介绍 Chrome 浏览器的架构，如下图所示。

在顶部，是「Browser Process」在和负责应用中其他部分的进程间进行协调。对于「Renderer Process」，会有多个进程被创建，并且分配到每个 tab 上。到目前为止，Chrome 尽可能给每个 tab 一个「Renderer Process」；而现在，它尝试给每个站点一个单独的「Renderer Process」，包括「iframes」。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210817231840763.png" alt="image-20210817231840763" style="zoom:80%;" />

## 进程控制什么？

下面的表格描述了每个 Chrome 进程，以及它们控制什么。

| 进程 | 进程是什么？有什么用？ |
| ---- | ---- |
| Brower Process | 控制浏览器的「chrome」部分，包括地址栏、书签栏、前进和后退的按钮。<br />同时也控制一些不可见的部分，比如 Web 浏览器的网络请求和文件访问的权限部分。 |
| Renderer Process | 控制 tab 中的网站显示。 |
| Plugin Process | 控制被网站使用的所有插件。比如，flash。 |
| GPU Process | 在独立于其他进程的情况下处理 GPU 的任务。它被分割成独立的进程是因为 GPU 处理来自多个应用程序的请求，并绘制在同一个平面上。 |

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210817233606779.png" alt="image-20210817233606779" style="zoom:80%;" />

还有更多的进程，比如「Extension Process」和「Utility Process」。如果你想了解有多少个进程运行在 Chrome 浏览器上，进入 Chrome 的任务管理器，可以看到 Chrome 当前运行的所有进程和它们消耗的 CPU/Memory 资源。

## Chrome 多进程架构的优点

在前面，我提到了 Chrome 使用了多个「Renderer Process」。这里举一个最简单的例子，你可以想象，每一个 tab 都有一个独立的「Renderer Process」。假设你有 3 个 tab，每个 tab 都运行在独立的「Renderer Process」。如果一个 tab 无响应，那么你可以关闭这个无响应的 tab，同时保持其他的 tab。而如果所有的 tab 都运行在同一个「Renderer Process」，那么当一个 tab 无响应，其他所有的 tab 都会变得无响应。

将浏览器的工作多个进程的另一个优点是其安全性和沙盒化。因为操作系统提供了一种限制浏览器权限的方法，浏览器可以将任意功能的任意进程沙盒化。比如，Chrome 浏览器限制了能处理任意用户输入的进程（如渲染进程）对于任意文件访问的权限。

因为进程有它们私有的内存空间，它们通常都有通用设施的拷贝（比如 V8 引擎）。这就意味着更多的内存消耗，因为它们无法像单进程多线程那样共享内存空间。为了节省内存空间，Chrome 限制了进程的数量。限制的数量取决于设备的内存和 CPU，但是当 Chrome 到达了这个上限，它开始在一个进程中运行来自同一站点的多个 tab。

## 节省过多的内存 —— Servicification in Chrome

> Servicification is "the migration from monolithic legacy application into service-based components"
>
> 对于 Chrome 来说，也就是 Splitting a monolithic Chrome browser up into components

同样的方法也适用于「Browser Process」。Chrome 正在进行架构上的改变，将浏览器的每个部分都作为一项服务来运行，允许被分割成不同的进程或者聚合成同一个进程。

通常的想法是，当 Chrome 运行在功能强大的硬件上时，它会将每个服务分割成不同的进程，提供更好的稳定性。但如果是在资源有限的设备上运行，Chrome 会将多个服务聚合成一个进程来节省内存占用。在此之前，类似的合并进程以减少内存消耗的方式已经在 Android 平台上使用过。

## 每个 frame 都有一个「Renderer Process」 ——  Site Isolation (站点隔离)

『站点隔离』是 Chrome 最近推出的功能，它对每一个跨站点的 iframe 都运行一个单独的「Renderer Process」。我们已经讨论过每个 tab 都有一个「Renderer Process」的模型，这个模型允许跨站点的 iframe 运行在一个和其他站点共享内存空间的进程中。在同一个「Renderer Process」中同时运行 a.com 和 b.com 看起来是没有问题的。『同源策略』是 Web 的核心安全模型，它确保一个站点不能未经允许从另外一个站点获取数据。绕过这个策略是安全攻击的基本目标。进程隔离是分割不同站点最有效的方式。随着 Meltdown 和 Spectre 漏洞的发现，我们对于使用进程区分站点的需求变得更加明显。自从 Chrome 67 以来，站点隔离在桌面上默认启动，tab 上每一个跨站点的 iframe 都有一个隔离的「Renderer Process」。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210818001205978.png" alt="image-20210818001205978" style="zoom:80%;" />

启用『站点隔离』是一项多年的工程工作。『站点隔离』并不只是单纯地分配「Renderer Process」，它在根本上改变了 iframe 之间的通信方式。能在一个有着 iframe 的页面上打开『开发者工具』，意味着『开发者工具』做了一些幕后工作，使得让它看起来能和普通页面没有什么区别。即便是在页面上使用简单的 Ctrl + F 来查找，那也是在跨越多个「Renderer Process」在查找。这样你就你可以明白，为什么浏览器开发者说『站点隔离』的发布是一个重大里程碑。

## 回顾

在上面的描述中，我们已经介绍了浏览器架构的高级视图，并且介绍了多进程架构的好处。我们同样介绍了 「Servicification」和「Site Isolation」，这两个特性和 Chrome 浏览器多进程架构密不可分。在接下来，我将开始介绍为了显示一个网站，进程和线程之间发生了什么。

