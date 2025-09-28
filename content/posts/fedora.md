+++
date = '2025-09-25T16:10:32+08:00'
draft = false
title = '使用 Fedora 打造属于自己的办公系统'
categories= ["linux"]
+++

## 前言

在购买 MBP 之前，自己在大学时期使用的笔记本上面折腾过黑苹果，现在依然清晰记得笔记本配置 CPU 是 intel i5-4200H，4GB 内存，1TB 硬盘。苹果系统之所以流畅，比 Windows 系统稳定，操作系统也是针对硬件进行了定制化开发，所以折腾黑苹果的过程中，每一个硬件都会带来驱动不兼容的问题，例如：显示器花屏、触控板不灵、蓝牙无法工作、扬声器没声音等。

2019 年自己花了 ￥12679 购买了一台最低配的 Intel 版本的 MBP（MacBookPro），其中还打了 6 千多元的京东白条。拿到之后经过浅浅体验，感觉好极了，系统流畅，超清视网膜屏幕，多手势触控板。再也不用为了体验苹果系统而去折腾黑苹果了，但也没有理想的那么好，散热不行、五国语言（宕机）、品牌溢价过高。换到了第二份工作之后，公司配备了 M 系列芯片的 MBP，再一次让我认为 MBP 是非常值得购买的笔记本，加之官方降价。迄今也觉得 MBP 是非常值得购买的笔记本。

第三份工作，公司配备了一台 8C8GB 的台式机，自己将初始化的 Windows 10 升级至了 11。Windows 11 配合 WSL（windows subsystem for linux） 使用起来的能够满足日常的办公，类似与苹果系统的全局搜索、原生支持 Linux 系统、交互体验大幅度提升。虽然 WSL 到目前已经了解决 Windows 安装 Linux 基础软件的问题，但对个人而言，还是有很多无法丝滑使用的地方。网络，WSL 支持 NAT 与 Mirror 两种模式转发网络，本质上这也是也一个轻量级的虚拟机，在使用容器部署服务或暴露 WSL 中的端口时，都显得相对麻烦。如果要验证一些内核相关的特性，根本无法满足，如 ebpf。于是，我决定尝试一款 Linux 桌面系统，脑袋里面蹦出了 Ubuntu、Debian、RockyLinux、Fedora。经过简单思索，优先试装 Fedora。对其它发行版有着刻板印象，Ubuntu 臃肿、Debian 简陋、RockyLinux 定位服务器。

## Fedora 简介

Fedora 作为 RHEL 的上游发行版，技术更新迭代迅速，Linus 也将 Fedora 作为自己的开发系统，自己或多或少受了一定的影响，Linus 曾公开说过，自己只是希望找一个安装方便的系统，更专于程序的开发。我还是幻想和技术大牛使用同样的系统，在将来的某一天也会如他一般技术超群，。

Fedora 与其它的 Linux 桌面发现版大小异同，提供 Gnome、KDE、XFCE 等桌面镜像下载。因我安装 Fedora 41 时，官方仅将 Fedora Workstation（Gnome）作为主推的发行版，我就无脑的选择了此桌面系统。 升级到 Fedora 42 后，官方额外增加了 KDE 桌面，未提的桌面系统皆为特殊兴趣的镜像维护。

## 前提

没事别瞎折腾，觉得 Windows 和 Mac OS 差强人意，或想玩一点不一样，不妨一试。如果有将将其作为个人的生产力主机的打算，务必提前确认使用自己办公的软件是否兼容。以下是我日常使用的一些软件：

| 名称 | 分类 | 安装方式 | 兼容度 |备注|
| :--- | :--- | :--- | :--- |:---|
| 微信 | 社交 | [安装包](https://linux.weixin.qq.com/)|高| |
| 飞书 | 办公|[安装包](https://www.feishu.cn/download)|高| |
| 钉钉 | 办公 |[faltpak](https://flathub.org/en/apps/com.dingtalk.DingTalk)|一般| |
| onlyoffice | 办公 | [flatpak](https://flathub.org/en/apps/org.onlyoffice.desktopeditors) |高|平替 Microsoft Office 文档|
|fcitx5|输入法|[flatpak](https://flathub.org/en/apps/org.fcitx.Fcitx5)|高 | |
|网易云音乐|媒体|不支持|网页 | |
|QQ音乐|媒体|不支持| 网页| |
|Clash Verge|网络|[安装包](https://github.com/clash-verge-rev/clash-verge-rev/releases)|高|科学上网|
|Podman|开发|[dnf](https://podman.io/docs/installation#fedora)|高|平替 Docker|
|Insomnia|开发|[安装包](https://github.com/Kong/insomnia/releases)|高|类似 Postman 的 API 管理工具|
|Notion|知识管理|不支持|网页||
|vscode|开发|[安装包](https://code.visualstudio.com/Download)|高||
|Chrome|浏览器|[安装包](https://www.google.com/chrome/)|高||
|QQ|社交|[安装包](https://im.qq.com/linuxqq/index.shtml)|高||

## 系统安装

### 准备

制作启动盘，推荐使用 [ventoy](https://www.ventoy.net/cn/index.html) 制作启动盘，可存储多个 ISO 文件至启动盘内，随机满足各种系统的装机需求。

显卡准备，不要建议使用 Nvidia 作为桌面显卡，nouveau 驱动适配的 Nvidia 显卡都是通过逆向完成的，Nvidia 可能会出现花屏、界面卡顿等问题。Linus 曾公开指责 Nvidia 官方，[Nvidia Fuck](https://www.youtube.com/watch?v=_36yNWw_07g)。没有 AMD 显卡，或不想购置的情况下，可参考大神的[文章](https://www.if-not-true-then-false.com/2015/fedora-nvidia-guide/)安装显卡驱动。

分区规划，使用桌面系统时，基本不会使用 root 账户，不用为 / 分区分配过多的容量，尽可能为 /home 分配多的磁盘空间，/boot 分配 1GB，/boot/efi分配 1GB。我一共有三块硬盘，分区如下：

| MOUNTED ON | SIZE | USED | AVAIL | TYPE | FILESYSTEM |
| :--- | :--- | :--- | :--- | :--- | :--- |
| / | 110.2G | 25.2G | 83.9G | btrfs | /dev/sda3 |
| /boot | 973.4M | 499.8M | 406.4M | ext4 | /dev/sda2 |
| /boot/efi | 571.1M | 19.3M | 551.8M | vfat | /dev/sda1 |
| /home | 465.8G | 146.0G | 317.3G | btrfs | /dev/nvme0n1p1 |
| /var | 111.8G | 16.7G | 93.5G | btrfs | /dev/sdb1 |

## 配置

### GNOME

Gnome 自定义程度较高，动手能力强的人可以尝试自己编写 gnome 扩展。实用插件推荐：

- [AppIndicator and KStatusNotifierItem Support](https://extensions.gnome.org/extension/615/appindicator-support/) 在任务栏展示应用程序图标。
- [Blur my Shell](https://extensions.gnome.org/extension/3193/blur-my-shell/) 毛玻璃化显示组件。
- [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/) 记录粘贴板历史
- [Quake Terminal](https://extensions.gnome.org/extension/6307/quake-terminal/) 全局快捷呼出应用程序，类似于 [iTerm2](https://iterm2.com/) 热键的功能。
- [Vitals](https://extensions.gnome.org/extension/1460/vitals/) 在任务栏显示各项系统监控信息。
- [Volume Scroller](https://extensions.gnome.org/extension/5904/volume-scroller/) 鼠标放置任务栏时，可通过滚轮调节音量大小。
- [Weather O'Clock](https://extensions.gnome.org/extension/5470/weather-oclock/) 在任务栏中的时钟旁显示天气信息。

### 输入法

1. 安装 [fcitx5](https://fcitx-im.org/wiki/Install_Fcitx_5)

    ```bash
    sudo dnf install fcitx5 fcitx5-autostart fcitx5-chinese-addons fcitx5-configtool fcitx5-gtk fcitx5-gtk2 fcitx5-qt
    ```

2. 配置输入法
   - 启用拼音输入法（Input Method 界面），将拼音添加至默认分组。
   - 调整拼音输入法（Input Method 界面），选中 Pinyin，然后点击设置按钮，调整 Page size 为 9；调整 Cloud Pinyin Index 为 2；新增向前翻页按键 `[`,向后翻页按键 `]`
   - 调整输入法切换快捷键（Global Options 界面），点击 `Trigger Input Method:` 栏的新增按钮(+)，然后录入 `Left shift` 按键。
   - 启用云拼音（Addons 界面），进入 Cloud Pinyin 详情页，设置 Minimum Pinyin Length 为 `4` 设置 Backend 为 `Baidu`。
