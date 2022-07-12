---
title: Ubuntu 安装踩坑记录
tags: []
categories: [记录]
date: 2022-07-11 17:35:51
mathjax:
description: 记录 Windows 和 Ubuntu 双系统的安装，以及 Ubuntu 的美化和配置
---

![image-20220712162842874](https://maples-ubuntu.oss-cn-hangzhou.aliyuncs.com/images/image-20220712162842874.png)

# 1 Ubuntu 和 Windows 双系统安装

目标：将 Ubuntu 20.04 装入移动硬盘中，不影响本地硬盘中 Windows 的使用。

设备：256G 移动固态硬盘，8G U盘

## 1.1 安装步骤

1. 使用 Rufus 和 Ubuntu 20.04 镜像将 U 盘制作为 Ubuntu 启动盘；
2. 从 U 盘启动中启动系统，开始 Ubuntu 系统的安装；
3. 硬盘分区，具体分区参见 1.2 节；
4. 启动项修复，将 Ubuntu 系统的启动项放在移动硬盘上，这样不影响 Windows 系统的启动；
5. 更换软件源。

## 1.2 硬盘分区

Linux 系统不同于 Windows 系统，前者是分区挂载在目录上，后者是目录挂载在分区上。我的 Ubuntu 系统分区如下：

| 分区       | 大小 | 主分区/逻辑分区 | 说明                   |
| ---------- | ---- | --------------- | ---------------------- |
| EFI 引导区 | 1 GB | 逻辑分区        | 用于 efi 引导          |
| SWAP 分区  | 8 GB | 主分区          | 与电脑的内存大小相匹配 |
| / 分区     | Rest | 逻辑分区        | 根目录挂载分区         |

> 注：在 Ubuntu 安装过程中，“安装启动引导器的设备”选择 EFI 引导区。

## 1.3 启动项修复

Ubuntu 系统安装完成后，重启电脑后会进入 grub 命令行界面，即便拔掉移动硬盘还是会进入 grub 命令行界面，具体原因还未探究，不过可以通过以下方法修复。

1. 重启电脑，进入 bios 界面选择从 Ubuntu 启动盘启动，进入后选择体验 Ubuntu；
2. 打开终端，安装 boot-repair 工具，`sudo add-apt-repository ppa:yannubuntu/boot-repair && sudo apt-get update`，`sudo apt-get install -y boot-repair && boot-repair`；
3. 进入 boot-repair 界面后，点击“recommended repair”，等待一段时间，显示修复成功。

重启电脑，选择从移动硬盘启动，即可进入 grub 选择菜单，选择 Ubuntu 就可以打开 Ubuntu 系统；断开移动硬盘，Windows也可正常启动。

## 1.4 更换软件源

进入 Ubuntu 设置界面，选择 About -> Software Update -> Download from，选择阿里云镜像。

# 2 Clash 配置

## 2.1 安装 Clash

安装步骤：

1. 安装 [Clash Core](https://github.com/Dreamacro/clash)；
2. 将 Clash 设置为守护进程，参考 [clash on a daemon](https://github.com/Dreamacro/clash/wiki/clash-on-a-daemon)。

命令：

1. 设置为开机启动：`systemctl enable clash`
2. 启动 clash：`systemctl start clash`
3. 关闭 clash：`systemctl stop clash`
4. 查看 clash 运行状态：`systemctl status clash`
5. 查看 clash 运行日志：`vim etc/clash/clash.log`
6. 拉取订阅服务器：`cd /etc/clash && sudo wget https://xxxxx && sudo mv xx.yaml config.yaml`

## 2.1 设置 System Proxy

1. 进入设置界面，选择 Network -> Network Proxy -> Mannual；
2. 设置如下图所示。

![image-20220712162640052](https://maples-ubuntu.oss-cn-hangzhou.aliyuncs.com/images/image-20220712162640052.png)

## 2.2 设置 Chrome Proxy

Chrome 安装 SwitchyOmega 插件，选择 System Proxy。

## 2.3 设置 Terminal Proxy

- 设置 proxy：`export HTTP_PROXY=http://127.0.0.1:7890;export HTTPS_PROXY=https://127.0.0.1:7890;export ALL_PROXY=socks5://127.0.0.1:7891`
- 取消 proxy：`unset HTTP_PROXY HTTPS_PROXY ALL_PROXY`

可以在 .zshrc 中设置 alias，方便启动：

```bash
alias setproxy='export HTTP_PROXY=http://127.0.0.1:7890;export HTTPS_PROXY=https://127.0.0.1:7890;export ALL_PROXY=socks5://127.0.0.1:7891'
alias unsetproxy='unset HTTP_PROXY HTTPS_PROXY ALL_PROXY'
```

# 3 zsh、oh-my-zsh 和 vim-airline 配置

zsh 和 oh-my-zsh 配置：

1. 安装 zsh 和 oh-my-zsh；
2. 设置 zsh 为默认 terminal：terminal -> preference -> profiles -> Command -> Custom command 设置为 zsh
3. 在 .zshrc 中设置主题为 agnoster；
4. 安装并设置 SourceCodePro 字体；
5. 启用设置：`source ~/.zshrc`。

vim airline 配置：

1. Vim 配置 [Vundle 插件管理器](https://github.com/VundleVim/Vundle.vim)；
2. 使用 Vundle 安装 [airline 插件](https://github.com/vim-airline/vim-airline)；
3. 下载 [powerline 字体](https://github.com/powerline/fonts)；
4. 下载 [vim-fugitive 插件](https://github.com/tpope/vim-fugitive)，此处可以直接 clone GitHub仓库到插件所在的文件夹，避免身份验证；
5. 在 .vimrc 中配置，相关配置如下。

> 使用 vim-fugitive 插件是为了在 airline 上显示 git 分支相关信息。

```
set nocompatible
filetype off
set laststatus=2

set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

Plugin 'VundleVim/Vundle.vim'

Plugin 'tpop/vim-fugitive'

Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'
let g:airline_theme="molokai"
let g:airline_powerline_fonts=1
set guifont="Source Code Pro for Powerline"
let g:airline#extensions#branch#enabled=1
			
call vundle#end()
filetype plugin indent on
```

效果如下：

![image-20220712162555206](https://maples-ubuntu.oss-cn-hangzhou.aliyuncs.com/images/image-20220712162555206.png)

# 4 软件安装

## 4.1 appimage 软件安装

参考 [Ubuntu下为AppImage应用添加图标并添加到应用](https://zhuanlan.zhihu.com/p/215507075)。

## 4.2 搜狗拼音输入法

坑：安装完成后无法输入中文。

原因：Ubuntu 20.04 取消了 qt 的相关依赖，需要自己安装。

解决方案：

`sudo apt install libqt5qml5 libqt5quick5 libqt5quickwidgets5 qml-module-qtquick2`

`sudo apt install libgsettings-qt1`

> 参考 https://pinyin.sogou.com/linux/help.php

## 4.3 vscode

- 官网下载 deb 文件安装，不要在 Ubuntu 软件商店安装，否则会无法输入中文；

- 设置同步：在 Windows 中登录 vscode 账号并将设置传到云端，在 Ubuntu 中登录账号同步云端设置。

## 4.4 typora

- dev 版：https://typora.io/windows/dev_release.html
- upload image 设置：使用 PicGo 上传图床

## 4.5 PicGo

使用 PicGo + 阿里云 OSS 设置图床。

## 4.6 Zotero

目标：Zotero + Onedrive 跨平台同步附件。

参考：[Zotero 跨平台同步附件的实现](https://zhuanlan.zhihu.com/p/31453719)，使用方法 3

关键：

1. Zotero Preferences -> Linked Attachment Base Directory：设置为 OneDrive 同步文件夹中子文件夹，且使用相对路径；

2. Zotero Preferences -> Data Directory Location：保持默认设置即可；

3. ZotFile Preferences -> Source Folder for Attaching New Files：设置为浏览器下载目录，用于 Attach New File功能；

4. ZotFile Preferences -> Location of Files：和 1 中路径相同。

![image-20220711190232319](https://maples-ubuntu.oss-cn-hangzhou.aliyuncs.com/images/image-20220711190232319.png)

## 4.7 dropbox

参考：[通过命令行安装无外设模式的 Dropbox](https://www.dropbox.com/install-linux)

## 4.8 OneDrive

GitHub 项目地址: [abraunegg/onedrive](https://github.com/abraunegg/onedrive)

安装教程：[Linux(Ubuntu)安装onedrive](https://zhuanlan.zhihu.com/p/343293386)

## 4.9 其它

1. Chrome
2. idea
3. wps
4. wemeet

# 5 开发环境安装

1. java
2. python
3. conda
4. node.js

# 6 系统美化

相关链接：

- [GNOME Shell Extensions](https://extensions.gnome.org/)
- [Gnome-look.org](https://www.gnome-look.org/browse/)

## 6.1 系统主题美化和 plank 安装

参考：[Ubuntu 20.04 桌面美化](https://zhuanlan.zhihu.com/p/176977192)

- 顶部栏显示软件图标：Tweaks 安装 Topicons plus 扩展；勾选 Ubuntu appindicators
- Appearance 中 Shell 报错：安装 User themes 扩展
- 剪贴板：安装 Clipboard indicator 扩展

## 6.2 GRUB 美化

参考：[Linux Grub引导界面（启动界面）美化](https://zhuanlan.zhihu.com/p/94331255)

# 7 Ubuntu 和 ios 传输文件

1. [Docs Transfer QR login](https://docstransfer.com/)
2. [Send Anywhere - File transfer](https://send-anywhere.com/#transfer)

# 8 其他

1. windows 和 ubuntu 时间同步问题：https://www.zhihu.com/question/46525639
2. Windows 迁移到新的移动硬盘：Windows 无法从移动硬盘中启动

# Reference

1. [ArchWiki](https://wiki.archlinux.org/)









