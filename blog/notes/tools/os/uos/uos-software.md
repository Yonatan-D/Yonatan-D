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

