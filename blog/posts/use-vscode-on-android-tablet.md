# 在安卓平板（Galaxy Tab S7）上使用 vscode

{docsify-my-updater}

## 起因

外出的时候想处理点简单的事情，比如码字、刷题、看代码等使用场景，背着笔记本就太笨重了，于是折腾了一番怎么把平板改造成生产力工具(？玩具！)。目前在用 Galaxy Tab 的 DeX 模式搭配 Termux 已经初步满足我的一些想法，配了个罗技 k380 蓝牙键盘，带出门这个重量还能接受。下面讲讲，我在尝试安装 vscode 过程中遇到的一些坑...

## 踩坑过程

为了极致的轻量和干净，我首先想的是使用浏览器版 vscode，采用 alpine + docker + code-server 的方案

termux 安装 alpine 和 docker，参考的这个视频：[如何在非 root 的手机上安装 docker](https://www.bilibili.com/video/BV1Fe4y127EJ)

code-server 官方镜像：https://hub.docker.com/r/codercom/code-server

之前我也有在用 code-server，已经习惯了这么使用：对于短暂使用的场景，我会起一个新容器，用完就销毁；长期的，多个容器彼此间的系统环境互不干扰（指使用终端，比如安装 nodejs、python）；如果有常用的系统环境也可以再构建一个定制化镜像。这一套组合已经很能打了，要说缺点的话，就是有些快捷键不起作用、有些扩展安装不是很方便。

当然，不玩 docker，仅安装浏览器版 vscode，只需要一行命令安装：

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

本着追求更极客的体验，我又整了 manjaro 和图形化界面（第一次体验 Arch 系），并安装 linux 版 vscode，于是开始踩坑

### 首先遇到安装命令不对

网上找到的教程均是 `pacman -Sy visual-studio-code-bin`，但我这报错找不到包，我察觉到可能是社区更新了，尝试了换源、添加 archlinuxcn 源也不能解决。后面用 `pacman -Ss vscode` 检索，发现包名应该是 `code`，正确的安装命令是：

```sh
$|pacman -Sy code
```

### 安装阶段继续报错 pgp signature 无效或损坏

似乎是签名的 key 出现了问题，找了很久解决方案，**以下命令对我不管用**：

```bash
pacman -Syy && pacman -S archlinuxcn-keyring
```

还尝试过使用 snap 安装 vscode 的方式，然后发现 termux 没有 systemctl 启动不了 snapd 服务，傻眼了

于是我放弃使用包管理工具了

### 选择从源码构建

先切换到非管理员用户进行操作（root 用户不能执行 makepkg 命令，可以新建一个拥有最高权限的普通用户），构建步骤如下：

```bash
curl -L -O https://aur.archlinux.org/cgit/aur.git/snapshot/visual-studio-code-bin.tar.gz
tar -xvf visual-studio-code-bin.tar.gz
cd visual-studio-code-bin
makepkg -si
```

安装成功。

### 但打开 vscode 无响应

在终端输入 code 命令是有信息输出的：

```bash
It is recommended to start vscode as a normal user.
To run as root, you must specify an alternate user data directory with the --user-data-dir argument.
```

它告诉我要用普通用户启动 vscode，无语...，按照它的提示，编辑快捷图标的启动命令为：

```bash
/usr/bin/code --no-sandbox --unity-launch
```

为了使 code 命令也能正常启动，可以在 .bashrc 加一条 alias

**至此，终于可以使用 vscode 了！**


### Process completed (signal 9)

另外，在使用 VNC 连接图形界面过程中，经常发生进程奔溃，我查到此问题会发生在 Android 12 下的 Termux。参考自 [Termux "Process completed (signal 9)" 错误解决方式](https://www.bilibili.com/video/BV1aZ4y1C73E)。（也可以看我的另一篇文章[通过无线调试解决 Termux “Process completed (signal 9)”](/posts/resolving-termux-signal-9-with-wireless-debugging)）

解决方法：

1. 下载 [Android SDK Platform-Tools](https://developer.android.com/studio/releases/platform-tools?hl=zh-cn)，并解压

2. 设备连接到电脑，打开 USB 调试模式（Tab S7 找到设置 —— 开发者选项 —— 调试 —— USB 调试，开启）

下面这条命令可以确认是否连接上设备，unauthorized 意味着未连接，device 即连接成功

```sh
$|./adb.exe devices
```

3. 在 platform-tools 目录内打开 cmd，执行以下命令

```sh
$|./adb.exe shell "/system/bin/device_config put activity_manager max_phantom_processes 2147483647"
```

短期内就不会遇到 signal 9 报错了，不过每次重启都必须执行这条命令

## 办公实践

分享下我在使用的应用程序：

- Office：WPS

- 浏览器：Kiwi Browser（基于 Chromium 开发，界面功能和 Chrome 非常相似，且支持从 Chrome 商店安装扩展）、Chrome（可惜不支持安装扩展，对比电脑版的功能阉割太多）、GitHub（客户端的网络很流畅）

- 远程桌面：RVNC Viewer，ToDesk，向日葵，TeamViewer（局域网使用）

- 编程：Termux（本文就是基于这个程序，是很好的学习和应急环境，但不能取代日常开发），QuickEdit（文本编辑器）

- 视频剪辑：不咕剪辑（我没剪过视频，但正在找一款能在平板上简单剪辑的 app，如果你有好用的欢迎推荐）

- 影音娱乐：哔哩哔哩、哔哩哔哩HD（在酷安下载）

其它未提到的，大多是用自带的应用程序，三星笔记很好用！
