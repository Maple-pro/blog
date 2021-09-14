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

导航提交后，「Renderer Process」继续加载资源和渲染页面。我们会在下篇文章中介绍在这个阶段中发生的细节。当「Renderer Process」完成渲染后，它会向「Browser Process」发送一个 IPC（这是在页面上所有 frame 中的 `onload` 事件触发并完成执行后发生的）。此时，「UI Thread」停止 tab 上的加载标志。

当然，这里的完成渲染并不是彻底停止了渲染，因为客户端的 JavaScript 仍然能加载额外的资源和渲染新的视图。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210914193753706.png" alt="image-20210914193753706" style="zoom:80%;" />

## 导航到其他站点

最简单的导航就已经结束了！但是当用户又重新在地址栏中输入了其他的 URL 时，发生了什么呢？「Browser Process」会通过相同步骤导航到其他站点。但是在这之前，它需要确认当前渲染的站点是否在意 `beforeunload` 事件。

当你尝试导航到其他站点或者关闭 tab 时，`beforeunload` 可以创建『是否确认离开该网站？』的提示。tab 中的所有东西，包括 JavaScript，都是由「Renderer Process」处理的，所以当新的导航请求传入时，「Browser Process」需要和当前的「Renderer Process」进行确认。

>注意：不要添加无条件的 `beforeunload` 处理事件。它会产生延迟，因为处理事件需要在导航开始前执行。只有在需要的时候才应该添加这个事件处理程序，比如警告用户可能会丢失页面上的输入的数据。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210914195627709.png" alt="image-20210914195627709" style="zoom:80%;" />

如果导航是从「Renderer Process」中启动的（比如用户点击了一个链接或者客户端的 JavaScript 代码执行了 `window.location = "https://newsite.com"` ），那么「Renderer Process」会首先检查 `beforeunload` 事件。然后，经历和「Browser Process」相同的过程。唯一的区别就是，导航请求是从「Renderer Process」到「Browser Process」开始启动的。

当导航到一个不同于当前的站点时，一个单独的「Renderer Process」会被调用来处理这个新的导航，而当前的「Renderer Process」会被保留用来处理类似于 `unload` 的事件处理程序。要了解更多，请看 [an overview of page lifecycle states](https://developers.google.com/web/updates/2018/07/page-lifecycle-api#overview_of_page_lifecycle_states_and_events)，以及如何使用 [the Page Lifecycle API](https://developers.google.com/web/updates/2018/07/page-lifecycle-api)。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210914200751458.png" alt="image-20210914200751458" style="zoom:80%;" />

## Service Worker

上述导航过程最近的一个变化是引入了 [service worker](https://developers.google.com/web/fundamentals/primers/service-workers)。Service worker 是在你的应用中编写网络代理的一种方式，它允许 web 开发者能更好地控制在本地缓存什么和什么时候从网络上获取新的数据。如果 service worker 设置为从缓存中加载页面，那么就不会发送请求从网络中获取数据。

需要记住的是，service worker 是运行在「Renderer Process」中的 JavaScript 代码。那么，当一个导航请求传入时，「Browser Process」怎么知道这个站点有 service worker 呢？

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210914201718678.png" alt="image-20210914201718678" style="zoom:80%;" />

当一个 service worker 注册时，service worker 的作用域会作为一个引用被保存（你可以在这篇 [The Service Worker Lifecycle](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle) 的文章中阅读更多有关 service worker 作用域的知识）。当一个导航发生后，「Network Thread」检查当前域名和注册的 service worker 的作用域，如果在这个 URL 中注册过，「UI Thread」会找到一个「Renderer Process」来执行 service worker 中的代码。这个 service worker 可能会从缓存中加载数据，消除从网络中请求数据的需要，或者也可能从网络中请求新的资源。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210914202452299.png" alt="image-20210914202452299" style="zoom:80%;" />

## 导航预加载

我们可以看到，如果 service worker 最终决定从网络中请求数据，那么会在「Browser Process」和「Renderer Process」中出现往返，造成延迟。[导航预加载](https://developers.google.com/web/updates/2017/02/navigation-preload) 就是一种通过在 service worker 启动时并行加载资源来加速这个过程的机制。它用 header 标记了这些请求，允许服务器决定为这些请求发送不同的内容，比如只是更新数据而不是整个文档。

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210914203030763.png" alt="image-20210914203030763" style="zoom:80%;" />

