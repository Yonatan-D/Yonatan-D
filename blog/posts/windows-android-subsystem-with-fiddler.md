# Windows 11 安装 Android 子系统并使用 Fiddler 抓包教程

{docsify-my-updater}

碎碎念... 如今高版本 Android 抓包没有 iOS 方便，需要获取 root 权限，再安装 xposed 或 Magisk 将 Fiddler 证书添加到系统证书目录，才能抓包，而在这之前得先解 BL 锁。

因为不想对主力机做“破坏性”的操作，我探索过以下方案：

- 方案1：安卓模拟器（雷电、MuMu）

- 方案2：Linux 下的 Waydroid（Arch 可看[此教程↗](https://wiki.archlinuxcn.org/wiki/Waydroid)）

- 方案3：Windows 下的 Android 子系统（Windows Subsystem for Android，简称 WSA）

- 方案4：基于 WSA 的腾讯应用宝（已接入微软商城，左侧工具栏多了个应用宝的导航，搜索 APP 图标上也有个应用宝的 logo）

除了方案1，我不想搞模拟器那么“重”和“割裂”，后面三种我都尝试了，方案3是我认为目前最好用的解决方案。

## 下面是我摸索出来的操作步骤：

##### 1. 安装 Windows 11 安卓子系统

下载构建好的已 root 装有 Magisk 的安卓子系统压缩包，解压并双击执行 Run.bat 脚本，等待安装完成

下载地址：[WSA_2407.40000.4.0_x64_Release-Nightly-with-magisk-29.0.29000.-stable-NoGApps-NoAmazon.7z](https://github.com/MustardChef/WSABuilds/releases/download/Windows_11_2407.40000.4.0_LTS_7/WSA_2407.40000.4.0_x64_Release-Nightly-with-magisk-29.0.29000.-stable-NoGApps-NoAmazon.7z)


> 项目地址：https://github.com/MustardChef/WSABuilds

##### 2. 安装 WSA 工具箱

下载页：https://apps.microsoft.com/detail/9ppsp2mkvtgt?gl=CN&hl=zh-CN

##### 3. 安装 Fiddler 证书

进入 WSA 工具箱，点击 Android 设置，安全 -> 更多安全设置 -> 加密与凭据 -> 安装证书 -> CA 证书，安装准备好的 FiddlerRoot.cer 证书文件

##### 4. 安装 MoveCertificate 模块

进入 Magisk 应用，点击模块，从本地安装，安装 MoveCertificate 模块，安装成功后重启

下载地址：https://github.com/ys1231/MoveCertificate/releases

##### 5. 连接 adb 调试

更多信息可参阅[此教程](/notes/debug?id=_43-通过-usb-连接电脑-用-fiddle-抓包)

```bash
# 设置 fiddler 代理
adb reverse tcp:8888 tcp:8888
adb shell settings put global http_proxy 127.0.0.1:8888
adb shell settings put global https_proxy 127.0.0.1:8888

# 抓包结束后执行以下命令来清除代理，否则无法上网
adb shell settings put global http_proxy :0
adb shell settings put global https_proxy :0
```

现在就可以使用 fiddler 来抓包了。

> 参考教程（好文收藏！）：  
> [MustardChef/WSABuilds RECOMMENDED FIX](https://github.com/MustardChef/WSABuilds/issues/593#issuecomment-3172749449)  
> [Windows 11 安卓子系统安装教程](https://www.jianeryi.com/1326.html)  
> [WSA 2.0？应用宝root并安装Magisk、GMS谷歌商店教程](https://zhuanlan.zhihu.com/p/1927406354584282919)  
> [win11中wsa使用fiddler抓包（https）](https://blog.csdn.net/lswandt/article/details/121821915)  
> [miui12使用Magisk 把fidder证书添加到系统证书目录](https://github.com/ys1231/MoveCertificate/releases)  
