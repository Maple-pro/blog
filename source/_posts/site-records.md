---
title: WordPress 建站记录
date: 2021-03-13 18:11:00
tags: 
categories: [记录]
description: Record how I build my WordPress site.
---

## 域名和服务器选择

> 域名在[NameSilo](https://www.namesilo.com/)上买的，一年7刀（网上可以找到优惠码，可以优惠1刀）。服务器在[Vultr](https://	www.vultr.com)上买的，走的日本线路，最便宜的那种，一个月5刀。

这么简单的东西我居然也能踩坑，我也是服了。

> 首先，如果域名和服务器其中一个是在国内整的，那么就需要备案。So，要么都买国内的，要么都买国外的。国内速度快，不过需要备案，备案挺快的；国外的速度没那么快，不过可以用服务器来整点活，而且不需要备案。
> 国内的话服务器可以买阿里云、华为云、腾讯云、百度云的，他们有学生优惠，比如阿里云10元/月。国外服务器的话，可以网上搜，比如Vultr、Linode等。Vultr上新手注册可以领100刀，但是要从特定的网址那里注册，还有一些其他的优惠。我的服务器速度也不太快，就不推荐了。
> 之后就是把域名解析到IP地址上，服务器在哪里买的就在那里整，大概15分钟后就可以了。

## 远程连接服务器安装WordPress

下载安装Xshell，新建会话，填写用户名和密码等（在购买的服务器的详细信息上会有）。这样就远程连接到服务器了。

## 安装nginx， MySQL，PHP，WordPress

[教程](https://blog.csdn.net/qq874455953/article/details/81603241)

## WordPress插件

**Crayon Syntax Highlighter （Crayon语法显示）**：代码高亮显示

**Jetpack**：支持markdown，latex的输入，以及网站统计等强大的功能

**LaTeX2HTML**：支持latex的输入，这个插件的公式渲染会比较好

**Mammoth .docx converter**：支持将word文档作为文章输入

**ThirstyAffiliates**：快速生成短链接（不过我也没怎么用过）

**UpdraftPlus-备份/恢复**：备份和恢复网站，升级还能迁移网站 	

**WP Tabel Tag Gen**：输入表格

## 其他

**主题**：Kratos

**字体样式**：

```css
body{
    font-family: "Helvetica Neue",Helvetica,Arial,"Microsoft Yahei","Hiragino Sans GB","Heiti SC","WenQuanYi Micro Hei",sans-serif;
}
```

**表格样式**：

```css
table,th,td{
        border-collapse:collapse;
        border:1px solid black;
        margin:auto;
	      text-align:center;
        }
```

## 缺点

用的是WordPress模板，没有自己整活，缺少了那种感觉。

也没有弄清Linux系统、和网站前后端的东西。

以后还会用这个服务器整点其他活（这个坑我挖下了，看我什么时候填起来）

敬请期待！