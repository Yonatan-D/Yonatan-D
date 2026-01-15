{docsify-my-updater date:2025-02-25}

# 让 windows terminal 更好用

## 前言

教你搞掂 windows terminal + git bash 配置，获得更接近 linux 体验的终端环境，非 wsl。

这个方案缺点是终端启动速度慢，使用 zsh 后就更慢了。

## 配置步骤

1. 安装 git，将 `C:\Program Files\Git\bin` 配置到环境变量 `PATH` 中

2. 配置 git bash 到 windows terminal 中，添加新配置文件，名称为 `Git Bash`，命令行为

```cmd
D:\Software\Git\bin\bash.exe --login -i
```

图标为 `C:\Program Files\Git\mingw64\share\git\git-for-windows.ico`

> 设置为默认 Shell：设置 -> 启动 -> 默认配置文件 -> Git Bash

3. 安装 zsh。

下载 zsh-5.9-2-x86_64.pkg.tar.zst，解压后将 etc 和 usr 目录复制到 Git 安装目录，即 C:\Program Files\Git，选择覆盖

zsh 最新版下载地址: https://packages.msys2.org/packages/zsh?repo=msys&variant=x86_64

4. 安装 oh-my-zsh，安装命令为

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

5. 安装 zsh 插件，我选择的是 `zsh-autosuggestions`、`zsh-syntax-highlighting` 和 `zsh-z`，安装命令为

```bash
git clone --depth 1 https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
git clone --depth 1 https://github.com/zsh-users/zsh-syntax-highlighting.git $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
git clone --depth 1 https://github.com/agkozak/zsh-z $ZSH_CUSTOM/plugins/zsh-z
```

6. 修改 .zshrc 配置主题和插件

```bash
sed -e 's|^plugins=(git)|plugins=(git zsh-autosuggestions zsh-syntax-highlighting)|g' \
    -e 's|ZSH_THEME="robbyrussell"|ZSH_THEME="agnoster"|g' \
    -i ~/.zshrc \
    ~/.zshrc
```

7. 补全一些常用工具命令

对于缺失的一些 linux 命令，例如 wget、tree、dig，可以下载二进制安装包，解压获得 wget.exe、tree.exe、dig.exe，然后复制到 Git 安装目录的 bin 目录下，也可以复制到自定义的目录，然后配置环境变量

完整配置：https://github.com/Yonatan-D/dotfiles/tree/win_cmd_gitBash

## 后话

因为不能忍受启动速度太慢，我换到了 Alacritty + WSL。现在用在 Windows 上的一整套快捷操作方案采用的是 Listary + Alacritty + AltDrag + WSL + Multipass，可以做到 CTRL + ` 打开管理员CMD，CTRL + 1 打开 Arch(WSL)，CTRL + 2 打开 Ubuntu(Multipass)，双击 CTRL 调出搜索框，文件夹内快捷执行命令等等。

感兴趣的可以参考我的配置文件: https://github.com/Yonatan-D/dotfiles/tree/main/private_dot_config

> [Listary](https://www.listary.com/) 可以快速检索、打开文件和程序。  
[Alacritty](https://alacritty.org/) 是一个跨平台的终端模拟器。  
[AltDrag](https://stefansundin.github.io/altdrag/) 可以让鼠标拖动窗口，和 KDE 一样，按住 ALT 键并在窗口内的任意位置点击左键来拖动窗口到新的位置，按住 ALT 并在窗口内任意位置点击右键后拖动来轻松调整它的大小。  
> [Multipass](https://canonical.com/multipass) 是 Ubuntu 官方推出的虚拟机管理工具，可以快速创建和管理虚拟机。
