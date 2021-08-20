---
title: 深入理解现代 Web 浏览器 (Part 2)
date: 2021-08-21 00:21:25
tags: [浏览器]
categories: [前端]
mathjax: false
description: "当用户请求一个站点时，浏览器做了什么准备工作去渲染页面？<br/>翻译自：https://developers.google.com/web/updates/2018/09/inside-browser-part2"
---

# Part 2. 在浏览器导航（Navigation）过程中，发生了什么？

> Reference: https://developers.google.com/web/updates/2018/09/inside-browser-part2

让我们想象一个场景：你在浏览器中输入了一个 URL，然后浏览器从互联网上获取数据并且展示相应的页面。在这篇文章中，我们会重点关注用户请求站点和浏览器准备呈现页面的部分 —— 即导航。

## 一切起始于「Browser Process」

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210821003027963.png" alt="image-20210821003027963" style="zoom:80%;" />

就如我们在 Part 1 中所讲的那样，tab 外所有的东西都归「Browser Process」管。「Browser Process」中有一些线程，比如在浏览器中绘制按钮和输入框的「UI Thread」，处理网络堆栈从而从互联网中接收数据的「Network Thread」，控制文件访问的「Storage Thread」等等。当你在地址栏中敲入一个 URL 后，你的输入就会被「Browser Process」中的「UI Thread」所处理。

## 一个简单的导航示例

### Step 1. 处理输入

当用户开始在地址栏中输入时，「UI Thread」要问的第一件事就是，“这是一个搜索查询还是一个 URL？”。在 Chrome 中，地址栏同样也是一个搜索输入框，所以「UI Thread」需要去解析并且判断你是想使用搜索引擎，还是想访问一个网站。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210821003942708.png" alt="image-20210821003942708" style="zoom:80%;" />

### Step 2. 开始导航

当用户敲回车后，「UI Thread」启动一个 network call 来获取网站内容。在 tab 的左上角会显示加载符号。「Network Thread」会调用一些协议，比如 DNS 查找，同时为后续请求建立 TLS 连接。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210821004445637.png" alt="image-20210821004445637" style="zoom:80%;" />

此时，「Network Thread」可能会收到一个服务器重定向的请求头，比如 HTTP 301。这种情况下，「Network Thread」会和重定向的那个「UI Thread」通信。然后，发起另外一个 URL 请求。

### Step 3. 读取响应

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210821004813517.png" alt="image-20210821004813517" style="zoom:80%;" />

一旦响应体（有效负载）开始进入，「Network Thread」在必要时会查看流的头几个字节。 Content-Type 响应头告知了数据的类型，但由于这个头可能会丢失或错误，因此这里会做一个 MIME Type sniffing 的操作。这是一件非常棘手的工作，你可以参考[代码](https://cs.chromium.org/chromium/src/net/base/mime_sniffer.cc?sq=package:chromium&dr=CS&l=5)。你可以看看代码中的注释，来了解不同浏览器时如何处理 content-type/payload 的。

如果响应是一个 HTML 文件，那个下一步就会把数据传递给「Renderer Process」，但如果是一个 zip 文件或者其他的什么文件，那就意味着这是一个下载请求，所以需要将数据传递给下载管理器。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210821005626512.png" alt="image-20210821005626512" style="zoom:80%;" />

这里也是 [SafeBrowsing](https://safebrowsing.google.com/) 检查发生的地方。如果域名和响应数据和已知的恶意网站相匹配，「Network Thread」会弹出警告页面。

另外，[Cross Origin Read Blocking (CORB)](https://www.chromium.org/Home/chromium-security/corb-for-developers) 检查是为了确保敏感的跨站数据不会进入「Renderer Process」。

### Step 4. 找到「Renderer Process」

当所有的检查完毕后，「Network Process」确保了应该导航到请求的站点，「Network Thread」告诉「UI Thread」数据已经准备完毕。「UI Thread」会找到对应的「Renderer Process」去承担页面渲染的工作。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210821010342185.png" alt="image-20210821010342185" style="zoom:80%;" />

由于网络请求可能需要几百毫秒才能得到响应，这里有一个优化来加速这个过程。在 Step 2 中，当「UI Thread」向「Network Thread」发送一个 URL 请求时，它已经知道应该导航到哪个站点。此时「UI Thread」尝试主动寻找或启动响应的「Renderer Process」，这个过程和网络请求是并行的。按照上述方式，如果所有的都成功了，那么当「Network Thread」接收到数据时，「Renderer Process」就已经处于待机状态了。如果导航需要重定向到另外一个站点，那么这个处于待机状态的「Renderer Process」就不会被使用，而会使用别的「Renderer Process」。

### Step 5. 提交导航

现在，所有的数据和「Renderer Process」都准备好了，「Browser Process」会向「Renderer Process」发送 IPC 去提交这次导航。同时也会传递数据流，以便「Renderer Process」能持续接收 HTML 数据。一旦「Browser Process」收到了确认，确保已经提交到了「Renderer Process」，那么这次导航就结束了，然后开始文档加载阶段。

此时，地址栏会更新，同时 security indicator 和 站点设置的 UI 会反映这个新页面的站点信息。这个 tab 的 session history 会被更新，从而前进/后退键可以进入到刚才访问过的站点。为了方便在关闭 tab / 窗口后能恢复 tab / 窗口，这些 session history 会被存储到磁盘上。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210821012558366.png" alt="image-20210821012558366" style="zoom:80%;" />

### Extra Step. 初始加载完成

