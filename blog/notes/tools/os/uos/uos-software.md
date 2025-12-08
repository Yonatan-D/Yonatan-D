# 软件安装问题

## IDEA

### 1 启动闪退

报错信息：The SUID sandbox helper binary was found, but is not configured correctly. Rather than run without sandboxing I'm aborting now. You need to make sure that /opt/apps/idea/jbr/lib/chrome-sandbox is owned by root and has mode 4755.

修改 bin/idea.vmoptions 文件，添加以下内容：

```bash
-Dide.browser.jcef.sandbox.enabled=false
```

## VSCode

### 1 隐藏系统窗口标题栏

- Settings -> Window -> Title Bar Style -> custom

### 2 设置平滑光标

- Settings -> Text Editor -> Cursor -> Cursor Blinking -> smooth

- Settings -> Text Editor -> Cursor -> Cursor Smooth Caret Animation -> on

## 微信开发者工具

https://github.com/msojocs/wechat-web-devtools-linux

## 支付宝：小程序开发者工具

应用商店能搜到wine版本，离线安装看这篇 [《下载离线安装包及依赖》](/notes/tools/uos/uos-faq?id=下载离线安装包及依赖)

### 1 降版本

1.1 先从应用商店安装并启动一次程序，让 wine 生成相关文件到 /opt/apps/com.alipay.devtools.deepin 目录

1.2 准备 windows 版本的 2.7.2 安装包：MiniProgramstudio-2.7.2.exe

1.3 在安装包目录下执行命令：

```bash
WINEPREFIX=$HOME/.deepinwine/com.alipay.devtools.deepin deepin-wine8-stable MiniProgramstudio-2.7.2.exe
```

> 这条命令也可以安装其它应用：WINEARCH=win32或者wine64 WINEPREFIX=容器路径 wine 需安装/运行的exe软件的路径

> 支付宝的安装包是64位的，也可以不传 WINEARCH 参数，会自动识别

1.4 删掉新的桌面快捷方式，用回旧的

### 2 卸载删除

```bash
rm -rf $HOME/.deepinwine
rm -rf $HOME/.wine
```

### 3 解决应用打不开的一些思路

3.1 在终端运行启动命令，查看运行日志。启动命令可以用文本编辑器打开桌面快捷方式查看

- `Exec="/opt/apps/com.alipay.devtools.deepin/files/run.sh" -f %f`

3.2 查看应用日志

```bash
cat $HOME/.deepinwine/com.alipay.devtools.deepin/drive_c/users/$USER/AppData/Roaming/小程序开发者工具/log/volans-log-日期/main.log
```

3.3 禁用GPU渲染

修改 $HOME/.deepinwine/com.alipay.devtools.deepin/drive_c/users/$USER/.mini-ide/settings.json
添加：`"core.disableGPU": true`

## 离线安装 NVM (Node.js)

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

## JDK

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
