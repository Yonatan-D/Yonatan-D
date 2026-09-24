# UOS（统信系统）

## 软件安装问题

### IDEA

#### 1 启动闪退

报错信息：The SUID sandbox helper binary was found, but is not configured correctly. Rather than run without sandboxing I'm aborting now. You need to make sure that /opt/apps/idea/jbr/lib/chrome-sandbox is owned by root and has mode 4755.

修改 bin/idea.vmoptions 文件，添加以下内容：

```bash
-Dide.browser.jcef.sandbox.enabled=false
```

### VSCode

#### 1 隐藏系统窗口标题栏

- Settings -> Window -> Title Bar Style -> custom

#### 2 设置平滑光标

- Settings -> Text Editor -> Cursor -> Cursor Blinking -> smooth

- Settings -> Text Editor -> Cursor -> Cursor Smooth Caret Animation -> on

### 微信开发者工具

https://github.com/msojocs/wechat-web-devtools-linux

### 支付宝：小程序开发者工具

应用商店能搜到wine版本，离线安装看这篇 [《下载离线安装包及依赖》](/notes/tools/uos/uos-faq?id=下载离线安装包及依赖)

#### 1 安装其它版本

1.1 先从应用商店安装并启动一次程序，让 wine 生成相关文件到 /opt/apps/com.alipay.devtools.deepin 目录

1.2 准备 windows 版本的 3.9.61 安装包：MiniProgramStudio-3.9.61-x64.exe

1.3 在安装包目录下执行命令：

```bash
WINEPREFIX=$HOME/.deepinwine/com.alipay.devtools.deepin deepin-wine8-stable MiniProgramStudio-3.9.61-x64.exe
```

> 这条命令也可以安装其它应用：WINEARCH=win32或者wine64 WINEPREFIX=容器路径 wine 需安装/运行的exe软件的路径

> 支付宝的安装包是64位的，也可以不传 WINEARCH 参数，会自动识别

1.4 删掉新的桌面快捷方式，用回旧的

#### 2 卸载删除

```bash
rm -rf $HOME/.deepinwine
rm -rf $HOME/.wine
```

#### 3 解决应用打不开的一些思路

3.1 在终端运行启动命令，查看运行日志。启动命令可以用文本编辑器打开桌面快捷方式查看

- `Exec="/opt/apps/com.alipay.devtools.deepin/files/run.sh" -f %f`

3.2 查看应用日志

```bash
cat $HOME/.deepinwine/com.alipay.devtools.deepin/drive_c/users/$USER/AppData/Roaming/小程序开发者工具/log/volans-log-日期/main.log
```

3.3 禁用GPU渲染

修改 $HOME/.deepinwine/com.alipay.devtools.deepin/drive_c/users/$USER/.mini-ide/settings.json
添加：`"core.disableGPU": true`

### 离线安装 NVM (Node.js)

1. nvm 离线安装步骤：

先准备好 nvm 安装包：

https://github.com/nvm-sh/nvm/archive/refs/tags/v0.39.7.tar.gz

```bash
# 新建 nvm 目录
sudo mkdir /usr/local/nvm
sudo chown -R admin:admin /usr/local/nvm

# 解压到 nvm 目录
sudo tar xzf v0.39.7.tar.gz –-strip-components=1 -C /usr/local/nvm

# 添加环境变量
sudo deepin-editor /etc/profile.d/nvm.sh

# 在 /etc/profile.d/nvm.sh 文件中添加以下内容（配置了默认版本为18.19.0，因为此时还没装node，所以请忽略报错）
export NVM_DIR="/usr/local/nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
nvm alias default v18.19.0 >> /dev/null

# 在 ~/.bashrc 文件末尾加上 source /etc/profile
sed -i '$a source /etc/profile' ~/.bashrc

# 重启终端，运行 nvm 命令，查看是否安装成功
nvm -v
```

2. node.js 离线安装步骤：

先准备好 node.js 安装包：

https://nodejs.org/download/release/v18.19.0/node-v18.19.0-linux-x64.tar.gz

```bash
# 在 nvm 安装目录下新建一个文件夹，用于存放 node
sudo mkdir -p /usr/local/nvm/versions/node

# 解压 node.js 安装包到 node 文件夹
sudo tar xzf node-v18.19.0-linux-x64.tar.gz -C /usr/local/nvm/versions/node/ --transform s/node-v18.19.0-linux-x64/v18.19.0/g
```

3. 使用 NVM 切换 Node 版本

```bash
# 显示当前可使用的 node 版
nvm ls

# 设置当前 node 版本为 18
nvm use v18.19.0
```

### JDK

```bash
# 下载并解压
wget https://repo.huaweicloud.com/java/jdk/8u151-b12/jdk-8u151-linux-x64.tar.gz
sudo tar -zxvf jdk-8u151-linux-x64.tar.gz -C /usr/local/
sudo mv /usr/local/jdk1.8.0_151 /usr/local/jdk1.8

sudo vim /etc/profile
# 添加以下内容
export JAVA_HOME=/usr/local/jdk1.8
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

source /etc/profile

# 检查是否生效
java -version
javac -version
```

## 问题解决

### vmware 安装 uos，实现与宿主机之间的复制粘贴

```bash
sudo apt install open-vm-tools-desktop -y

# 如果提示找不到软件包, 先执行下 sudo apt-get update
```

### 离线环境本地编译wine

不推荐，速度慢，且需要网络执行 fix-broken，建议是直接下载个 wine 版软件就有 deepin-wine8-stable

```bash
# 下载源码:
https://dl.winehq.org/wine/source/8.0/wine-8.0.2.tar.xz

# 需要补全的依赖：
flex: http://archive.ubuntu.com/ubuntu/pool/main/f/flex/flex_2.6.4-6.2_amd64.deb
bison: http://archive.ubuntu.com/ubuntu/pool/main/b/bison/bison_3.5.1+dfsg-1_amd64.deb
xserver-xorg-dev: http://archive.ubuntu.com/ubuntu/pool/main/x/xorg-server/xserver-xorg-dev_1.20.13-1ubuntu1~20.04.12_amd64.deb

apt --fix-broken install

# 编译：
./configure --enable-win64
make
make install
```

### 下载离线安装包及依赖

-d 仅下载，不安装

```bash
sudo apt clean
sudo apt install -d -y com.alipay.devtools.deepin

mkdir alipayDevtools
cp /var/cache/apt/archives/*.deb alipayDevtools
```

拷贝到离线机上，执行安装命令：

```bash
sudo dpkg -i alipayDevtools/*.deb
```

### wine 缺少中文字体

从 C:\Windows\Fonts\ 复制宋体字体（simsun.ttc）到 uos 桌面，双击安装

### 桌面图标

```toml
[Desktop Entry]
Version=1.0
Name=
Exec=
StartupNotify=true
Terminal=false
Icon=
Type=Application
Categories=
```

- StartupNotify - 双击启动时鼠标的转圈动画
- Terminal - 设置 false 静默执行，不弹出终端窗口

### 关闭浏览器跨域限制

chrome 内核浏览器均可以设置 --disable-web-security 关闭安全策略，解决跨域

```sh
Exec=/usr/bin/google-chrome-stable --disable-web-security --user-data-dir=/home/username/.config/google-chrome-dev
```

统信浏览器需要多去掉 X-Deepin 前缀的 3 行配置

### nodejs项目启动报错：Error: ENOSPC: System limit for number of file watchers reached

这是因为文件监视程序的系统产生了限制，达到了默认的上限，需要增加限额

```bash
echo fs.inotify.max_user_watches = 524288 | sudo tee -a /etc/sysctl.conf 
sudo sysctl -p
```

### arm架构下 node 编译的 canvas、images

1. canvas: [No prebuilt found for arm64 ](https://github.com/Automattic/node-canvas/issues/1662)

自己编译：

```bash
$ sudo apt-get update 
$ sudo apt-get install build-essential libcairo2-dev libpango1.0-dev libjpeg-dev libgif-dev librsvg2-dev
$ yarn add canvas
```

2. images: [images 没有适配ARM64？](https://github.com/zhangyuanwei/node-images/issues/240)

直接下载编译好的：

```bash
https://github.com/zhangyuanwei/node-images/files/7709901/linux-arm64-binding.node.zip
```
