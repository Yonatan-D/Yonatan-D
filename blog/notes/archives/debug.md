# 问题排查

## Android调试技巧

adb工具下载：https://googledownloads.cn/android/repository/platform-tools-latest-windows.zip

### 查看第三方应用包名

```shell
adb shell pm list packages -3
```

### 查看uniapp的运行日志

```shell
adb shell "logcat | grep 'console :'"
```

### 设置fiddle代理进行抓包

背景：不具备让电脑和手机连入同一网络的条件，当前电脑连接的是手机热点，该方法是通过usb调试将手机上的8888端口映射到电脑上的8888端口，然后电脑使用fiddle进行抓包

1. 将手机上的 8888 端口映射到电脑上的 8888 端口

```cmd
adb reverse tcp:8888 tcp:8888
```

2. 手机访问 http://localhost:8888 下载CA证书，导入证书：设置 - 安全 - 更多安全设置 - 从手机存储安卓 - CA证书

```cmd
adb shell settings put global http_proxy 127.0.0.1:8888
adb shell settings put global https_proxy 127.0.0.1:8888
```

3. 抓包结束后，必须执行以下命令来清除代理，否则手机将无法上网

```cmd
adb shell settings put global http_proxy :0
adb shell settings put global https_proxy :0
```

### 使用脚本快速过滤某一进程的日志

环境：windows下使用wsl去执行bash脚本，adb工具在windows下安装

```bash
#!/bin/bash
APP_PACKAGE=$1
PROCESS_INFO=$(adb.exe shell ps | grep $APP_PACKAGE)
if [ -z "$PROCESS_INFO" ]; then
    echo "应用 $APP_PACKAGE 未运行"
    exit 1
fi
IFS=$'\n' read -d '' -r -a PROCESSES <<< "$PROCESS_INFO"

echo "找到 ${#PROCESSES[@]} 个进程："
echo "序号 | 进程号 | 进程名称"
echo "------------------------"
for i in "${!PROCESSES[@]}"; do
    PID=$(echo "${PROCESSES[$i]}" | awk '{print $2}')
    PROCESS_NAME=$(echo "${PROCESSES[$i]}" | awk '{print $9}')
    printf "%-4s | %-6s | %s\n" "$((i+1))" "$PID" "$PROCESS_NAME"
done

echo

while true; do
    read -p "请输入要抓取日志的进程序号 (1-${#PROCESSES[@]}): " choice
    
    if ! [[ "$choice" =~ ^[0-9]+$ ]]; then
        echo "请输入有效的数字"
        continue
    fi
    
    if [ "$choice" -lt 1 ] || [ "$choice" -gt "${#PROCESSES[@]}" ]; then
        echo "请输入 1 到 ${#PROCESSES[@]} 之间的数字"
        continue
    fi
    
    break
done

SELECTED_PID=$(echo "${PROCESSES[$((choice-1))]}" | awk '{print $2}')
SELECTED_PROCESS_NAME=$(echo "${PROCESSES[$((choice-1))]}" | awk '{print $9}')
echo "已选择进程：$SELECTED_PROCESS_NAME (进程号: $SELECTED_PID)"

echo "开始抓取 $APP_PACKAGE 的日志..."

# 开始抓取日志
adb.exe logcat --pid=$SELECTED_PID
```