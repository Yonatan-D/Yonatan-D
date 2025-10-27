# 问题解决

## vmware 安装 uos，实现与宿主机之间的复制粘贴

```bash
sudo apt install open-vm-tools-desktop -y

# 如果提示找不到软件包, 先执行下 sudo apt-get update
```

## 离线环境本地编译wine

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

## 下载离线安装包及依赖

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

## wine 缺少中文字体

从 C:\Windows\Fonts\ 复制宋体字体（simsun.ttc）到 uos 桌面，双击安装

## 桌面图标

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

## 关闭浏览器跨域限制

chrome 内核浏览器均可以设置 --disable-web-security 关闭安全策略，解决跨域

```sh
Exec=/usr/bin/google-chrome-stable --disable-web-security --user-data-dir=/home/username/.config/google-chrome-dev
```

统信浏览器需要多去掉 X-Deepin 前缀的 3 行配置