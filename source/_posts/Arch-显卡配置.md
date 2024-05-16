---
title: Arch 显卡配置
tags: []
categories: [记录]
date: 2024-05-12 00:26:09
mathjax:
description:
---

前段时间电脑经常出现关机/重启时整个界面卡住，只能按住电源键强制重启的情况，到 [Arch Forum 发帖](https://bbs.archlinux.org/viewtopic.php?pid=2170412)后，发现是 NVIDIA 显卡驱动的问题，需要降级到 535xx。在降级的过程中发现，之前安装 Intel 显卡和 NVIDIA 显卡驱动时，安装了过时的驱动（博客害人，还是要看 wiki），因此这里将整个显卡驱动安装梳理一遍。

![](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20240515-27e223-20240515155739.png)

<!-- more -->

# 1. Arch 显卡驱动安装

## 1.1 Intel 显卡驱动

参考文档：

- [Wiki: Intel graphics installation](https://wiki.archlinux.org/title/Intel_graphics#Installation)
- [Wiki: OpenGL](https://wiki.archlinux.org/title/OpenGL)
- [Wiki: Vulkan](https://wiki.archlinux.org/title/Vulkan)

安装：

- mesa: OpenGL drivers
- lib32-mesa: OpenGL drivers (32-bit)
- mesa-utils: Essential mesa utilities
- vulkan-intel: Vulkan driver for Intel GPUs
- lib32-vulkan-intel: Vulkan driver for Intel GPUs (32-bit)

> 对于 Gen 11 及以上的 CPU，由于 xf86-video-intel 缺少对 11 代及以上 CPU 的支持，因此不要安装 xf86-video-intel，安装后可能导致界面卡死的情况。
> ![](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20240515-a4bb0f-20240515161849.png)

## 1.2 NVIDIA 显卡驱动

参考文档：
- [Wiki: NVIDIA](https://wiki.archlinux.org/title/NVIDIA)

最新版本安装：

- nvidia: NVIDIA drivers for linux (or nvidia-open)
- nvidia-utils: NVIDIA drivers utilities, include OpenGL implementation and Vulkan driver
- lib32-nvidia-utils: NVIDIA drivers utilities (32-bit), include OpenGL implementation and Vulkan driver
- opencl-nvidia: OpenCL implementation for NVIDIA
- lib32-opencl-nvidia: OpenCL implementation for NVIDIA
- nvidia-settings: Tools for configuring the NVIDIA graphics driver
- nvidia-prime (extra): NVIDIA Prime Render Offload configuration and utilities

特定旧版本安装（aur）：

- nvidia-535xx-dkms: replacement for nvidia package
- nvidia-535xx-utils
- lib32-nvidia-535-utils
- opencl-nvidia-535xx
- lib32-opencl-nvidia-535xx
- nvidia-535xx-settings
- nvidia-prime (extra)

## 1.3 Xorg 驱动安装

参考文档：

- [Wiki: Xorg driver installation](https://wiki.archlinux.org/title/Xorg#Driver_installation)

> 不要安装 xf86-video-intel, xf86-video-nouveau, xf86-video-vesa，而是使用 Xorg 默认的 modesetting。

## 1.4 NVIDIA 显卡驱动降级

[Forum: Repeated kernel problems/freezes since 6.7.6](https://bbs.archlinux.org/viewtopic.php?id=293400)

Arch Forum 中 [这篇帖子](https://bbs.archlinux.org/viewtopic.php?id=293400&p=2) 指出，NVIDIA driver 550 版本存在问题，会导致关机或重启时界面冻结，需要降级到 535 版本。降级方式直接从 aur 中安装 535xx 版本即可，需要移除对应的最新版本驱动。

> 安装 nvidia-535xx-settings 时，需要降级依赖 libxnvctrl，即需要先安装 libxnvctrl-535xx (aur)。

```
# 降级的 pkglog
--------------------------------------------------------------------------------
2024-05-11 16:09:07 xf86-video-vesa    2.6.0-2 removed
2024-05-11 16:09:07 xf86-video-nouveau 1.0.17-3 removed
2024-05-11 16:09:07 xf86-video-intel   1:2.99.917+923+gb74b67f0-2 removed
2024-05-11 16:09:07 libxvmc            1.0.14-1 removed
--------------------------------------------------------------------------------
2024-05-11 16:16:48 xorg-xbacklight 1.2.3-3 removed
2024-05-11 16:18:19 light-git       1.2.2.r6.gd717f1c-1 installed
--------------------------------------------------------------------------------
2024-05-11 16:36:42 patchelf 0.18.0-3 installed
2024-05-11 16:36:43 dkms     3.0.12-1 installed
2024-05-11 16:38:54 dkms     3.0.12-1 reinstalled
--------------------------------------------------------------------------------
2024-05-11 16:43:06 opencl-nvidia             550.78-1 removed
2024-05-11 16:43:06 nvidia                    550.78-2 removed
2024-05-11 16:43:06 lib32-opencl-nvidia       550.78-1 removed
2024-05-11 16:43:06 lib32-nvidia-utils        550.78-1 removed
2024-05-11 16:43:06 nvidia-utils              550.78-1 removed
2024-05-11 16:43:06 nvidia-535xx-utils        535.179-1 installed
2024-05-11 16:43:07 lib32-nvidia-535xx-utils  535.179-1 installed
2024-05-11 16:43:07 lib32-opencl-nvidia-535xx 535.179-1 installed
2024-05-11 16:43:07 nvidia-535xx-dkms         535.179-1 installed
2024-05-11 16:43:07 opencl-nvidia-535xx       535.179-1 installed
2024-05-11 16:44:25 libxnvctrl                550.78-1 reinstalled
--------------------------------------------------------------------------------
2024-05-11 16:49:54 libxnvctrl                  550.78-1 removed
2024-05-11 16:49:54 libxnvctrl-535xx            535.179-1 installed
2024-05-11 16:50:01 nvidia-settings             550.78-1 removed
2024-05-11 16:50:01 nvidia-535xx-settings       535.179-1 installed
2024-05-11 16:50:01 nvidia-535xx-settings-debug 535.179-1 installed
--------------------------------------------------------------------------------
```

## 1.5 视频硬件加速

参考文档：

- [Wiki: Hardware video acceleration](https://wiki.archlinux.org/title/Hardware_video_acceleration)

安装：

- nvidia-utils

# 2. 双显卡配置

参考文档：

- [Wiki: NVIDIA Optimus](https://wiki.archlinux.org/title/NVIDIA_Optimus)
- [Optimus Manager Documentation](https://github.com/Askannz/optimus-manager)

## 2.1 optimus-manager 安装

> 如果是使用混合模式，则不必使用 optimus-manager

安装：

- optimus-manager
- bbswitch

配置：

- Switching method: Bbswitch
- Startup mode: Hybrid

## 2.2 prime-run 方法

参考文档：

- [Nvidia GPU offloading for "hybrid" mode](https://github.com/Askannz/optimus-manager/wiki/Nvidia-GPU-offloading-for-%22hybrid%22-mode)

命令：

安装 prime-run

```bash
prime-run %command%
```

# 3. 常用命令

```bash
# 生成 initramfs
sudo mkinitcpio -P

# 生成 grub.cfg
sudo grub-mkconfig -o /boot/grub/grub.cfg

# 查看 Xorg 驱动
xrandr --listproviders

# 查看 OpenGL render（需要安装 mesa-utils）
glxinfo | grep "OpenGL render"
prime-run glxinfo | grep "OpenGL render"

# 查看 NVIDIA 显卡是否被禁用，如果为 rev ff，说明被禁用
lspci | grep NVIDIA
# 01:00.0 VGA compatible controller: NVIDIA Corporation AD107M [GeForce RTX 4060 Max-Q / Mobile] (rev a1)
# 01:00.1 Audio device: NVIDIA Corporation Device 22be (rev a1)

# 显示显卡驱动
lspci -k | grep -A 2 -E "(VGA|3D)"
# 00:02.0 VGA compatible controller: Intel Corporation Raptor Lake-S UHD Graphics (rev 04)
#         Subsystem: Lenovo Device 3b53
#         Kernel driver in use: i915
# --
# 01:00.0 VGA compatible controller: NVIDIA Corporation AD107M [GeForce RTX 4060 Max-Q / Mobile] (rev a1)
#         Subsystem: Lenovo Device 3b53
#         Kernel driver in use: nvidia

# 测试显卡性能
glxgears
glxgears -info | grep GL_RENDERER
# GL_RENDERER   = Mesa Intel(R) Graphics (RPL-S)
prime-run glxgears -info | grep GL_RENDERER
# GL_RENDERER   = NVIDIA GeForce RTX 4060 Laptop GPU/PCIe/SSE2

# 查看 OpenGL 驱动情况
eglinfo -B

# 查看当前日志
jounalctl -xef

# 查看上一次启动的日志文件，并上传
sudo journalctl -b -1 | curl -F 'file=@-' 0x0.st
sudo journalctl -r -b -1 # 倒序

# 列出所有活动的 service
systemctl list-unit-files --state=enabled

# show intel gpu usage
sudo intel_gpu_top

# show all gpu usage
nvtop

# show nvidia infomation
nvidia-smi
```

# 4. SysRq 配置和使用

参考文档：[Wiki: Kernal SysRq](https://wiki.archlinux.org/title/Keyboard_shortcuts#Kernel_(SysRq))

配置：

- 新建 sysctl 配置文件 `/etc/sysctl.d/99-sysctl.conf`
- 添加配置：`kernel.sysrq = 1`

使用：

![](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20240515-4db948-20240515171441.png)

> 当界面卡住无响应时可以使用（内核 dump 时无法使用），SysRq 键即为 PrtSc 键。

- 重启：`Alt-SysRq-b`（避免 hard reboot）
- 触发 OOM：`Alt-SysRq-f`

# 5. memtest86+

对内存做压力测试。

参考文档：[Wiki: Stress testing - MemTest86+](https://wiki.archlinux.org/title/Stress_testing#MemTest86+
)

安装：

- 安装 memtest86+-efi 包
- 重新生成 grub.cgd

启动：

- 从 GRUB 的 memtest 选项中进入

# 6. 其他

## 6.1 将 xorg-xbacklight 改为 light

参考文档：[Wiki: backlight](https://wiki.archlinux.org/title/backlight)

原因：xorg-xbacklight 不适用于默认的 modesettings 驱动。

```bash
sudo pacman -Rs xorg-xbacklight
yay -S light
```

## 6.2 yakuake 外接屏幕不居中的问题

在 AutoStart 中添加环境变量：`env QT_SCREEN_SCALE_FACTORS="1.2;1.2" yakuake`

> 由于 QT 升级到了 qt6，原本的 `1;1` 失效，需要将原本的 `1;1` 变为 `1.2;1.2`。
