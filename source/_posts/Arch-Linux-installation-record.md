---
title: Arch Linux 安装配置记录
tags: []
categories: [记录]
date: 2023-03-28 18:10:14
mathjax:
description:
---

![20230504143256](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/20230504143256.png)

本文记录了 Arch Linux 和 Windows 11 双系统安装和基本配置的流程。

- 硬件：Intel CPU，Nvidia GPU，双硬盘（nvme0n1 & nvme1n1）；
- 软件：Windows 11（安装在 nvme0n1 上），启动类型为 UEFI；
- 目标：将 Arch Linux 安装在 nvme1n1 上，开机时通过 GRUB 引导选择启动系统。

<!-- more -->

# 1. Arch Linux 安装

## 1.1 参考

安装过程主要参考了以下几篇博客和官方 Wiki：

- [Wiki: Installation guide](https://wiki.archlinux.org/title/Installation_guide)
- [Wiki: Dual boot with Windows](https://wiki.archlinux.org/title/Dual_boot_with_Windows)
- [Blog: Arch Linux + Windows 双系统安装教程](https://blog.linioi.com/posts/18/)
- [Blog: 以官方 Wiki 的方式安装 Arch Linux](https://www.viseator.com/2017/05/17/arch_install/)

## 1.2 安装步骤

1. 关闭 Windows 上的 UEFI Secure Boot、Fast Startup 和 Hibernation；
2. 下载 Arch Linux 镜像，使用 Rufus 烧录 Arch Linux 镜像到 U 盘上；
   1. 注意分区类型选择 GPT，目标系统类型选择 UEFI；
3. 从 U 盘中启动 Arch 安装系统；
4. 连接网络
   1. 解除无线网卡的禁用：`rfkill unblock all`
   2. 进入 iwctl：`iwctll`
   3. 列出电脑中的无线网卡：`device list`，得到无线网卡名称为 `wlan0`
   4. 列出所有无线网络：`station wlan0 get-networks`
   5. 连接无线网络：`station wlan0 connect WIFI_NAME`
   6. 退出 iwctl：`exit`
   7. 检测是否连接成功：`ping baidu.com`
5. 更新系统时间
   1. `timedatectl set-ntp true`
6. 硬盘分区与格式化
   1. 本次安装仅划分两个分区：
      1. 一个是根分区，使用了 nvme1n1 上的所有空间，挂载在 `/` 目录下；
      2. 一个是启动分区，使用了 Windows 中的 EFI 分区，即 /dev/nvme0n1p1，不另外单独划分，挂载在 `/boot/` 目录下。
      3. 系统分区挂载后的结果如下（注：nvme1n1p1 分区为 Windows 划分，不用管）： 
          ```
          λ ~/ lsblk
          NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
          nvme1n1     259:0    0 931.5G  0 disk 
          ├─nvme1n1p1 259:1    0    16M  0 part 
          └─nvme1n1p2 259:2    0 931.5G  0 part /
          nvme0n1     259:3    0 953.9G  0 disk 
          ├─nvme0n1p1 259:4    0   260M  0 part /boot
          ├─nvme0n1p2 259:5    0    16M  0 part 
          ├─nvme0n1p3 259:6    0 951.6G  0 part 
          └─nvme0n1p4 259:7    0     2G  0 part 
          ```
   2. 划分根分区：`cfdisk /dev/nvme1n1`，将硬盘中所有空闲空间全部创建为根分区；
   3. 格式化根分区：`mkfs.ext4 /dev/nvme1n1p2`。
7. 挂载分区
   1. 挂载根分区：`mount /dev/nvme1n1p2 /mnt`
   2. 挂载启动分区：`mkdir /mnt/boot/`, `mount /dev/nvme0n1p1 /mnt/boot/`
8. 选择镜像源：
   1. 修改 `/etc/pacman.d/mirrorlist` 文件，在 Server 列表开头添加清华源和浙大源：
    ```
    Server = http://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
    Server = http://mirrors.zju.edu.cn/archlinux/$repo/os/$arch
    ```
   2. 刷新镜像源：`pacman -Syy`
9. 安装基本包：`pacstrap /mnt base base-devel linux linux-firmware vim e2fsprogs ntfs-3g networkmanager` 
10. 配置 Fstab
    1. 生成自动挂载分区的 fstab 文件：`genfstab -L /mnt >> /mnt/etc/fstab`
    2. 查看生成的文件是否正确：`cat /mnt/etc/fstab`
11. 进入新系统：`arch-chroot /mnt /bin/bash`
12. 设置时区：
    1.  `ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`
    2.  `hwclock --systohc`
13. 设置 Locale
    1.  `vim /etc/locale.gen`，注释 en_US, zh_CN 两行；
    2.  `locale-gen`
    3.  `vim /etc/locale.conf`，在第一行添加 `LANG=en_US.UTF-8`
14. 设置 hostname 和 root 密码
    1. 设置 hostname:
       1. `echo arch > /etc/hostname`
       2. `vim /etc/hosts`，添加如下内容：
          ```
          127.0.0.1 localhost
          ::1 localhost
          127.0.1.1 arch.localdomain arch
          ```
    2. 修改 root 密码：`passwd`
15. 创建用户
    1. 创建用户：`useradd -m -G wheel -s /bin/bash yangfeng`
    2. 修改密码：`passwd yangfeng`
    3. 添加 root 权限：`vim /etc/sudoers`，取消 `# %wheel ALL=(ALL) ALL` 注释
16. 安装 Intel-ucode：`pacman -S intel-ucode`
17. 安装 Bootloader：
    1. 安装 os-prob、grub 和 efibootmgr：`pacman -S os-prober grub efibootmgr`
    2. 部署 grub：`grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub`
    3. 生成 grub 配置文件：`grub-mkconfig -o /boot/grub/grub.cfg`
    4. 检查 grub 配置文件：`vim /boot/grub/grub.cfg`
       1. 这里有可能没有检测到 Windows 的启动项，是因为目前所处的 U 盘安装系统无法检测到其他系统的入口，稍后重启进入 Arch 系统后再次生成 grub 配置文件即可
18. 退出 Arch 系统：`exit`
19. 取消挂载分区：`umount /mnt/boot`, `umount /mnt`
20. 重启：`reboot`

# 2. Arch Linux 配置

## 2.1 参考

配置过程主要参考了以下博客：

- [ArchLinux安装后的必须配置与图形界面安装教程](https://www.viseator.com/2017/05/19/arch_setup/)

## 2.2 配置过程

### 2.2.1 创建交换文件

1. 在前面的安装过程中，我没有创建交换分区，而是在这里使用交换文件
2. 分配一块空间用于交换文件：`dd if=/dev/zero of=/swapfile bs=1M count=8192 status=progress`
3. 更改权限：`chmod 600 /swapfile`
4. 设置交换文件：`swapon /swapfile`
5. 在 fstab 中为交换文件设置入口：`vim /etc/fstab`，在最后一行加上 `/swapfile none swap defaults 0 0`

### 2.2.2 图形界面安装

![20230329164517](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/20230329164517.png)

GNOME 安装：

1. 安装显卡驱动：`sudo pacman -S xf86-video-intel`
2. 安装 Xorg：`pacman -S xorg`
3. 安装 gnome 和 gdm-prime：`pacman -S gnome gdm-prime`
4. 设置 gdm-prime 开机自启：`systemctl enable gdm-prime`
5.  将 Wayland 切换为 Xorg：
    1. `vim /etc/gdm/custom.conf`，取消 `#WaylandEnable=false` 的注释
    2. 添加 `QT_QPA_PLATFORM=xcb` 到 `etc/environment` 中
    3. 检测当前是 Wayland 还是 Xorg：`echo $XDG_SESSION_TYPE`

![20230329144644](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/20230329144644.png)

KDE 安装：

1. 安装显卡驱动和 Xorg
2. 安装 KDE：`pacman -S plasma sddm`
3. 设置 sddm 开机自启：`systemctl enable sddm`

![20230401195811](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/20230401195811.png)

![20230504143057](https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/20230504143057.png)

> GNOME 切换为 KDE：`systemctl disable gdm-prime`


### 2.2.3 安装中文字体

安装字体：`pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji adobe-source-han-sans-otc-fonts`

### 2.2.4 配置网络

开机自启 NetworkManager：`systemctl enable NetworkManager`

### 2.2.5 pacman 配置

1. 添加 Arch Linux CN 源
   1. 在 `/etc/pacman.conf` 末尾添加如下内容：注意不要使用 tsinghua 源
       ```
       [archlinuxcn]
       Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
       ```
   2. 刷新源：`pacman -Syy`
   3. 安装密钥：`pacman -S archlinuxcn-keyring`
2. 开启 multilib
   1. 在 `/etc/pacman.conf` 中注释 multilib 对应的内容

### 2.2.6 安装 yay

`pacman -S yay`

### 2.2.7 显卡配置

1.  安装 Nvidia 显卡闭源驱动：`sudo pacman -S nvidia nvidia-prime nvidia-settings nvidia-utils opencl-nvidia lib32-nvidia-utils lib32-opencl-nvidia`
2.  安装双显卡切换工具：`yay -S optimus-manager optimus-manager-qt`
3.  安装 nouveau：`pacman -S xf86-video-nouveau`
4.  添加 `prime-offload` 到 `~/.xinitrc` 中

如何指定 Nvidia 显卡启动应用？

1. 指定环境变量：`__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia command args...`
2. 示例：`__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia glxgears -info | grep GL_RENDERER`
3. 设置 alias：`alias nvidia="__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia"`

## 2.3 应用安装

### 2.3.1 中文输入法

1. 参考：[在 Arch 上使用 Fcitx5](https://www.cnblogs.com/qscgy/p/13385905.html)
2. 安装 fcitx5：`pacman -S fcitx5-chinese-addons fcitx5-git fcitx5-gtk fcitx5-qt fcitx5-pinyin-zhwiki kcm-fcitx5`
3. 修改环境变量：
   1. 添加以下内容到 `~/.pam_environment`
     ```
     GTK_IM_MODULE DEFAULT=fcitx
     QT_IM_MODULE  DEFAULT=fcitx
     XMODIFIERS    DEFAULT=@im=fcitx
     ```
4. 默认启动 fcitx5：
   1. 添加 `fcitx5 &` 到 `~/.xprofile` 中
5. 配置主题：[Fcitx5-Material-Color](https://github.com/hosxy/Fcitx5-Material-Color)

### 2.3.2 clash

1. 安装 clash：`pacman -S clash`
2. 运行 Clash 生成配置文件：`clash`
   1. 配置文件位于 `~/.config/clash/` 下
3. [Run clash as a service](https://github.com/Dreamacro/clash/wiki/Running-Clash-as-a-service)
   1. Note: Clash binary is in `/usr/bin/clash`
   2. Clash configuration file is in `~/.config/clash/`
4. Set proxy in terminal:
   1. 将下面内容添加到 `~/.zshrc` 中
      ```
      alias setproxy='export HTTP_PROXY=http://127.0.0.1:7890;export HTTPS_PROXY=http://127.0.0.1:7890;export ALL_PROXY=socks5://127.0.0.1:7891'
      alias unsetproxy='unset HTTP_PROXY HTTPS_PROXY ALL_PROXY'

      alias setgitproxy='git config --global http.proxy "http://127.0.0.1:7890";git config --global https.proxy "http://127.0.0.1:7890"'
      alias unsetgitproxy='git config --global --unset http.proxy; git config --global --unset https.proxy'


      alias reclash='sudo systemctl restart clash'
      ```
5.  [Clash web ui](http://clash.razord.top/#/proxies)
6.  [Clash for windows bin](https://aur.archlinux.org/packages/clash-for-windows-bin)

### 2.3.3 Steam

1. 安装 Steam：`pacman -S steam`
2. Steam HiDPI 设置：在 `/usr/share/applications/steam.desktop` 上添加启动环境变量 `Exec=env GDK_SCALE=2 /usr/bin/steam-runtime %U`
3. [About desktop entry](https://wiki.archlinux.org/title/desktop_entries)

### 2.3.4 OneDrive

1. Install [OneDrive](https://aur.archlinux.org/packages/onedrive-abraunegg): `yay -S onedrive-abraunegg`
2. Run OneDrive: `onedrive --synchronize`

### 2.3.5 Printer

> Reference: [CUPS](https://wiki.archlinux.org/title/CUPS)

1. Install cups: `pacman -S cups`;
2. Install cups-pdf: `pacman -S cups-pdf`;
3. Start cups service: `systemctl enable cups; systemctl start cups`;
4. Using Avahi: `pacman -S nss-mdns; systemctl enable avahi-daemon.service; systemctl start avahi-daemon.service`
5. Install GUI application: `pacman -S system-config-printer`;
6. Open web interface: http://localhost:631/

### 2.3.6 flameshot

flameshot HiDPI 设置：

在 `/usr/share/applications/org.flameshot.Flameshot.desktop` 中修改 `Exec` 项为 `Exec=env QT_SCREEN_SCALE_FACTORS="1; 1" /usr/bin/flameshot`。

> 参考：https://wiki.archlinux.org/title/HiDPI#Qt_5

flameshot shortcuts 设置：

在 KDE 设置中的 Shortcuts 界面添加命令 `env QT_SCREEN_SCALE_FACTORS="1; 1" flameshot gui`，添加快捷键 `F1` 和 `PrtSc`。

### 2.3.7 其他

1. dolphin
2. konsole
3. bluetooth
4. Chrome
5. yakuake
6. linuxqq: 如何遇到崩溃退出，删除 `~/.config/QQ/`
7. deepin-wine-wechat
8. visual-studio-code-bin
9. idea: 安装 Window Decorate Mode Switcher 插件用于隐藏 Top Bar
10. telegram
11. [nvm](https://github.com/nvm-sh/nvm): Node Version Manager
12. flameshot
13. zsh & oh-my-zsh
14. nvtop
15. zotero (zotfile plugin)

## 2.4 系统美化

### 2.4.1 Extensions

GNOME:

- [AppIndicator and KStatusNotifierItem Support](https://extensions.gnome.org/extension/615/appindicator-support/)
- [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/)
- [Dash to Dock](https://extensions.gnome.org/extension/307/dash-to-dock/)
- [Hide Top Bar](https://extensions.gnome.org/extension/545/hide-top-bar/)

### 2.4.2 gnome-tweaks

1. 安装 gnome-tweaks：`pacman -S gnome-tweaks`
2. 安装美化包：
   1. [WhiteSur Gtk Theme](https://www.gnome-look.org/p/1403328/) -> `/usr/share/themes/`
   2. [WhiteSur icon Theme](https://www.pling.com/p/1405756/) -> `/usr/share/icons/`
   3. [McMojave cursors](https://www.pling.com/p/1355701/) -> `/usr/share/icons`
3. 将左上角的 activities 图标从 MacOS 更改为 Arch：
   1. 将 activities-arch 的图标放入 `/usr/share/themes/WhiteSur-Dark-solid/gnome-shell/assets` 中；
   2. 修改 `/usr/share/themes/WhiteSur-Dark-solid/gnome-shell/gnome-shell.css`，替换 activities 图标。

### 2.4.3 grub theme

- [grub2-themes](https://github.com/vinceliuice/grub2-themes)
- 如何设置 grub 默认启动项：修改 `/etc/default/grub` 文件

### 2.4.4 HiDPI

用户登录页面 HiDPI 缩放比设置：

参考：[How to force hiDPI scaling mode on boot in Pop!_OS/Gnome](https://unix.stackexchange.com/questions/530748/how-to-force-hidpi-scaling-mode-on-boot-in-pop-os-gnome)

1. Open the configuration file:
    `/usr/share/glib-2.0/schemas/org.gnome.desktop.interface.gschema.xml`
2. Change the default value to the scaling factor
    ```
    <key name="scaling-factor" type="u">
    <default>2</default>
    ```
3. Apply the change: `sudo glib-compile-schemas /usr/share/glib-2.0/schemas` 

> **KDE per-display scaling**
>
> [Reference](https://www.reddit.com/r/kde/comments/l9z7wp/how_to_enable_perdisplay_scaling/)
>
> 1. 将显示器设置为 Extent to right
> 2. `xrandr --output HDMI-0 2x2`
> 3. `xrandr --output HDMI-0 1.5x1.5`
>
> 或者直接使用 alias 中的 `monleft` / `monright`

### 2.4.5 KDE Theme

安装 [WhiteSur KDE Theme](https://github.com/vinceliuice/WhiteSur-kde)

- Application Style: Breeze
- Plasma Style: Arc Dark / WhiteSur-dark
- Colors: WhiteSurAlt / WhiteSur
- Window Decorations: Breeze / WhiteSur_x1.5
- Icons: WhiteSur
- Cursors: [McMojave Cursors](https://github.com/vinceliuice/McMojave-cursors)
- Widgets:
  - [Window title](https://aur.archlinux.org/packages/plasma5-applets-window-title)
  - Global Menu
  - Event Calendar

## 2.5 Shortcuts & alias (KDE)

- `Super + E`: open dolphin
- `Super + D`: hide all normal windows
- `F1`: flameshot gui
- `Super + i`: open setting
- `Super + tab`: switch activeties
- `alt + tab`: switch windows
- `Ctrl+F7`: Toggle present windows (Window Class)
- `Ctrl+F10`: Toggle present windows (All desktop)
- `Alt+Tab`: Switch between windows (All windows)
- ``Alt+` `` : Switch between windows (Current application)
- `Super+Tab`: Switch between workspace
- `Ctrl+F1`: Switch to first virtual desktop
- `Ctrl+F2`: Switch to second virtual desktop
- `Alt-Shift-F12`: toggle compositor

```bash
alias setproxy='export HTTP_PROXY=http://127.0.0.1:7890;export HTTPS_PROXY=http://127.0.0.1:7890;export ALL_PROXY=socks5://127.0.0.1:7891'
alias unsetproxy='unset HTTP_PROXY HTTPS_PROXY ALL_PROXY'

alias setgitproxy='git config --global http.proxy "http://127.0.0.1:7890";git config --global https.proxy "http://127.0.0.1:7890"'
alias unsetgitproxy='git config --global --unset http.proxy; git config --global --unset https.proxy'

alias reclash='sudo systemctl restart clash'

alias updateclash="sudo wget 'https://api.balala.top/net/xxxxxxxxxxxx?app=clash' -O /etc/clash/config.yaml"

alias tonvidia="prime-offload;optimus-manager --switch nvidia"

alias tointel="prime-offload;optimus-manager --switch integrated"

alias showgpu="glxinfo | grep 'OpenGL renderer'"

alias startyakuake="nohup yakuake > ~/Documents/log/yakuake.log &"
alias startoptimusqt="nohup optimus-manager-qt > ~/Documents/log/optimus-manager-qt.log &"

alias nvidia="__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia"

# Mount windows disk
alias mountwin="sudo mkdir /run/media/yangfeng/Windows-SSD;sudo mount -o rw /dev/nvme0n1p3 /run/media/yangfeng/Windows-SSD"
alias umountwin="sudo umount -v /run/media/yangfeng/Windows-SSD;sudo rm -r /run/media/yangfeng/Windows-SSD"

# Start wine in zh_CN
alias chwine="env LANG=zh_CN.UTF-8 wine"

# Extend external monitor to left/right, scale it to 1.5x1.5, and set the built-in monitor refresh rate to 60MHz
alias monleft="xrandr --output HDMI-0 --mode 1920x1080 --pos 0x0 --rotate normal --scale 2x2 --output eDP-1-1 --mode 2560x1600 --pos 2880x0 --rotate normal --rate 60.00;xrandr --output HDMI-0 --scale 1.5x1.5"
alias monright="xrandr --output eDP-1-1 --mode 2560x1600 --pos 0x0 --rotate normal --rate 60.00 --output HDMI-0 --mode 1920x1080 --pos 2560x0 --rotate normal --scale 2x2;xrandr --output HDMI-0 --scale 1.5x1.5"

# Restart KDE
alias restartkde="kquitapp5 plasmashell || killall plasmashell && kstart5 plasmashell &"
# restart kwin: kwin_x11 --replace
```

## 2.6 双系统相关问题

### 2.6.1 [Time standard](https://wiki.archlinux.org/title/Dual_boot_with_Windows#Time_standard)

### 2.6.2 [Bluetooth pairing](https://wiki.archlinux.org/title/Dual_boot_with_Windows#Bluetooth_pairing)

[bt-dualboot](https://github.com/x2es/bt-dualboot)

### 2.6.3 更新 Bios 后启动不显示 GRUB 界面

> https://my.oschina.net/u/4418120/blog/3575719

1. 使用 arch 启动盘
2. `mount /dev/nvme1n1p2 /mnt`
3. `mkdir /mnt/boot`
4. `mount /dev/nvme0n1p1 /mnt/boot`
5. `arch-chroot /mnt /bin/bash`
6. reinstall grub: `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=grub`
7. generate grub.cfg: `grub-mkconfig -o /boot/grub/grub.cfg`
8. `exit`
9. `umount /mnt/boot`
10. `umount /mnt`
11. reboot and select the arch system in the grub menu to boot
12. after boot the arch system, regenerate the grub.cfg: `sudo grub-mkconfig -o /boot/grub/grub.cfg`

Note (About grub):

- 操作系统菜单目录：/etc/grub.d/ 系统自动生成
- grub 配置文件：/etc/default/grub 用户配置文件，可以配置主题、默认启动项等
- grub 引导文件: /boot/grub/grub.cfg （grub-mkconfig 生成）

## 2.7 其他

### 2.7.1 日志

- pacman: `/var/log/pacman.log`
- xorg: `/var/log/Xorg.0.log`
- 系统日志：`journalctl -xe`
- 查询 PID 日志：`journalctl _PID=888`
- clash: 
  - `/etc/clash/log.log`
  - `systemctl status clash`

### 2.7.2 配置文件

- `~/.xinitrc`
- `~/.xprofile`
- `~/.pam_environment`

pacman 配置文件：

- `/etc/pacman.d/mirrorlist`
- `/etc/pacman.conf`

### 2.7.3 命令

- 不进入图形界面，进入命令行：`Ctrl-Alt-F2`
- optimus-manager
    ```
    prime-offload

    optimus-manager --switch nvidia
    optimus-manager --switch integrated

    # use glxinfo to show current GPU used
    glxinfo | grep "OpenGL render"
    ```
- `dmesg`: Write the kernel message to standard output
- `lscpu`: Display information about the cpu architecture
- `intel_gpu_top`: show intel gpu usage
- `nvtop`: show nvidia gpu usage
- `nvidia-smi`: show nvidia gpu usage
- `conky`: show all gpu usage

### 2.7.4 触摸板手势

1. 安装 [libinput-gestures](https://github.com/bulletmark/libinput-gestures) 和 xdotool：`yay -S libinput-gestures xdotool`
2. 配置自定义手势：`nvim ~/.config/libinput-gestures.conf`

```
# Switch to next desktop
gesture swipe right 4 xdotool key ctrl+F1
# Switch to prev desktop
gesture swipe left 4 xdotool key ctrl+F2

# Present windows (current desktop)
gesture swipe up 4 xdotool key ctrl+F9

# Show desktop
gesture swipe down 4 xdotool key super+d

# Pinch
gesture pinch in 2 xdotool key ctrl+minus
gesture pinch out 2 xdotool key ctrl+plus
```

四指左右滑动切换虚拟桌面，四指上滑查看所有窗口，四指下滑显示桌面，两指缩放。

### 2.7.5 字体配置

参考：

- [Linux fontconfig 的字体匹配机制](https://catcat.cc/post/2020-10-31/)
- [用 fontconfig 治理 Linux 中的字体](https://catcat.cc/post/2021-03-07/)

在 `~/.config/fontconfig/` 下添加配置文件 `fonts.conf`：[完整配置文件](https://gist.github.com/Maple-pro/8ffe02b11768ccd7291126b6cf2726d2)

### 2.7.6 关闭响声

运行命令 `xset -b`，可以将其设为启动脚本。

### 2.7.7 Failed to start Load/Save Screen Backlight Brightness

现象：

- 启动系统时出现 Failed to start Load/Save Screen Backlight Brightness of backlight:nvidia_0；
- 系统运行 `journalctl -xe` 出现 ACPI Error 报错信息。

解决方案：

- 运行命令：`sudo systemctl mask systemd-backlight@backlight:nvidia_0.service`

# 3. 相关文档和链接

Arch Linux Documentation:

- [Arch Wiki](https://wiki.archlinux.org/)
- [pacman](https://wiki.archlinux.org/title/pacman)
- [Arch Packages](https://archlinux.org/packages/)
- [AUR](https://aur.archlinux.org/)
- [Arch Linux 中文社区仓库](https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/)

其他：

- [Arch Linux 简明指南](https://arch.icekylin.online/)
- [Gnome-look](https://www.gnome-look.org/browse/)
- [Gnome Shell Extensions](https://extensions.gnome.org/)

