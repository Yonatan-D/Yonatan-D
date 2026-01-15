{docsify-my-updater date:2023-07-02}

# 在安卓平板（Galaxy Tab S7）上使用 vscode

?> 2026-01-04 更新：不追求 vscode 的体验，可以看下这个编辑器 [Xed-Editor](https://github.com/Xed-Editor/Xed-Editor)，使用下来感觉不赖。来自 HelloGitHub 的介绍：适用于 Android 的代码编辑器。这是一款开源的 Android 文本与代码编辑器，内置 Termux 终端可运行 Python 和 Node.js，支持 200+ 编程语言语法高亮、自动缩进和文件管理等功能。

## 起因

短期外出想用平板取代笨重的笔记本做轻度办公，折腾怎么把平板改造成~~生产力工具~~玩具，探索了一番 Android 上使用 Linux 环境及软件，最后确定 Galaxy Tab 的 DeX 模式 + Termux，搭配罗技 k380 蓝牙键盘的方案，带出门这个重量还能接受。下面是我在平板上探索如何安装 vscode 的过程。

## 踩坑过程

安装网页版 vscode，只需要一行命令：

```bash
curl -fsSL https://code-server.dev/install.sh | sh
```

为了将常用开发环境打包，我又搞了一套方案：alpine + docker + code-server

termux 安装 alpine 和 docker，参考的这个视频：[如何在非 root 的手机上安装 docker](https://www.bilibili.com/video/BV1Fe4y127EJ)

code-server docker镜像：https://hub.docker.com/r/codercom/code-server

由于网页版 vscode 快捷键不起作用、有些扩展安装不是很方便，我又装了桌面版 manjaro + vscode(linux)，下面是我遇到的一些坑...

### 安装命令不对

网上找到的教程均是 `pacman -Sy visual-studio-code-bin`，但我执行后报错找不到包，可能是社区更新了，换源、添加 archlinuxcn 源也不能解决。后面用 `pacman -Ss vscode` 检索，发现包名应该是 `code`，正确的安装命令是：

```sh
pacman -Sy code
```

### pgp signature 无效或损坏

似乎是签名的 key 出现了问题，找了很久解决方案，**以下命令对我不管用**：

```bash
pacman -Syy && pacman -S archlinuxcn-keyring
```

也尝试过使用 snap 安装 vscode，然后发现 termux 没有 systemctl 启动不了 snapd 服务。于是我放弃使用包管理工具了

### 选择从源码构建

先切换到非管理员用户进行操作（root 用户不能执行 makepkg 命令，可以新建一个拥有最高权限的普通用户），构建步骤如下：

```bash
curl -L -O https://aur.archlinux.org/cgit/aur.git/snapshot/visual-studio-code-bin.tar.gz
tar -xvf visual-studio-code-bin.tar.gz
cd visual-studio-code-bin
makepkg -si
```

安装成功。

### 双击 vscode 图标无法启动

在终端运行 code 命令能看到错误信息：

```bash
It is recommended to start vscode as a normal user.
To run as root, you must specify an alternate user data directory with the --user-data-dir argument.
```

不推荐用 root 用户启动 vscode。我选择继续用 root 用户，编辑快捷图标的启动命令修改为：

```bash
/usr/bin/code --no-sandbox --unity-launch
```

为了使 code 命令也能正常启动，可以在 .bashrc 加一条 alias 命令：

```bash
alias code="/usr/bin/code --no-sandbox --unity-launch"
```

### Process completed (signal 9)

另外，在使用 VNC 连接图形界面过程中，经常发生进程奔溃，我查到此问题会发生在 Android 12 下的 Termux。参考自 [Termux "Process completed (signal 9)" 错误解决方式](https://www.bilibili.com/video/BV1aZ4y1C73E)。（也可以看我的另一篇文章 [Termux "Process completed (signal 9)" 的免连接电脑解决办法](/posts/resolving-termux-signal-9-with-wireless-debugging)）
