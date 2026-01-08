# Termux "Process completed (signal 9)" 的免连接电脑解决办法

{docsify-my-updater date:2023-10-02}

该方法适用于 Android 11 及以上版本。无需连接电脑。

1. 在 termux 里安装 adb 工具：
```bash
pkg install android-tools
```
2. 在“开发者选项”里启用“无线调试”，进入“无线调试”点击“使用配对码配对设备”，获得IP、端口和配对码，回到 termux 填入配对码：
```bash
adb pair ip:port
```
3. 连接设备。进入“无线调试”，上面写着设备名称和IP端口，回到 termux 输入IP端口：
```bash
adb connect ip:port
```
4. 查看设备列表是否成功连接上，unauthorized 意味着未连接，device 即连接成功：
```bash
adb devices
```
5. 使用此命令停用 Phantom process killing：
```bash
adb shell "/system/bin/device_config put activity_manager max_phantom_processes 2147483647"
```
这样就不会遇到 signal 9 报错了，不过每次重启机器都要再执行这条命令。

参考自：[Termux "Process completed (signal 9)" 错误解决方式](https://www.bilibili.com/video/BV1aZ4y1C73E)
