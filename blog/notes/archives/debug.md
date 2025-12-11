# 问题排查

## 1 浏览器调试技巧

### 1.1 断点源码

F12 -> Source -> Ctrl + P 搜索文件 -> 设置断点

### 1.2 断点网络请求

步骤1：F12 -> Source -> XHR/fetch Breakpoints -> 添加需要拦截的请求地址，可以直接填域名

步骤2：触发请求进入断点后，可以从堆栈找到发起请求的文件，点击堆栈中的函数名可以跳转到源码处

### 1.3 请求重发（curl）

步骤1：F12 -> Network -> 选择需要重发的请求 -> 右键 -> Copy -> Copy as cURL

步骤2：打开命令行工具，粘贴cURL命令，然后回车，即可重新发起请求（或者导入 Postman）

### 1.4 模拟微信内置浏览器环境

步骤1：F12，打开 Network Conditions 面板

- 点击右上角三点 → More tools → Network conditions

- 取消勾选 Use browser default

步骤 2：输入微信 UA

```
MicroMessenger/6.0.0.54_r849063.5015
```

仅修改 UA 无法完全模拟微信内置浏览器环境，只能辅助排查一些问题

如果是公众号网页，可以使用微信开发者工具调试

## 2 在 VSCode 中进行浏览器调试

```json
{
    "name": "Launch Chrome",
    "type": "chrome",
    "request": "launch",
    "url": "http://localhost:3000",
    "webRoot": "${workspaceFolder}"
},
{
    "name": "Launch Edge",
    "request": "launch",
    "type": "msedge",
    "url": "http://localhost:3000",
    "webRoot": "${workspaceFolder}"
}
```

## 3 Nodejs调试技巧

### 3.1 简单调试JS文件

VSCode 开一个 JavaScript Debug Terminal，然后输入 `node JS文件路径`，就可以断点调试了。

推荐使用 `nodemon JS文件路径`，修改保存后可以自动重启。

```sh
npx nodemon JS文件路径
```

### 3.2 使用 launch.json 文件调试

VSCode 左侧 `Run and Debug` -> `create a launch.json file`，选择 `Node.js`，然后修改 `program` 字段为你的文件路径，就可以在 `Run and Debug` 中点击 `Start Debugging` 来调试了，或者按 `F5` 键启动调试。

### 3.3 VSCode 调试 Egg

https://github.com/atian25/blog/issues/25

## 4 Android调试技巧

adb工具下载：https://googledownloads.cn/android/repository/platform-tools-latest-windows.zip

### 4.1 查看第三方应用包名

```shell
adb shell pm list packages -3
```

### 4.2 查看uniapp的运行日志

```shell
adb shell "logcat | grep 'console :'"
```

### 4.3 设置fiddle代理进行抓包

背景：不具备让电脑和手机连入同一网络的条件，当前电脑连接的是手机热点，该方法是通过usb调试将手机上的8888端口映射到电脑上的8888端口，然后电脑使用fiddle进行抓包

步骤1：将手机上的 8888 端口映射到电脑上的 8888 端口

```cmd
adb reverse tcp:8888 tcp:8888
```

> Android 允许我们通过 ADB，把 Android 设备上的某个端口映射到电脑（adb forward），或者把电脑的某个端口映射到 Android 设备（adb reverse）

步骤2：手机访问 http://localhost:8888 下载 CA 证书并导入手机

vivo（OriginOS 6）：设置 - 安全 - 更多安全设置 - 从手机存储安卓 - CA证书

步骤3：设置代理，将手机上的所有请求都转发到电脑的 8888 端口

```cmd
adb shell settings put global http_proxy 127.0.0.1:8888
adb shell settings put global https_proxy 127.0.0.1:8888
```

步骤4：抓包结束后，必须执行以下命令来清除代理，否则手机将无法上网

```cmd
adb shell settings put global http_proxy :0
adb shell settings put global https_proxy :0
```

### 4.4 使用脚本快速过滤某一进程的日志

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

## 5 写一个请求代理工具

?> 解决我自己痛点的一个小工具，会考虑开源：[web-debugging-proxy-devtools](https://gitee.com/yonatan/web-debugging-proxy-devtools.git)【WIP】  


特性：

- 默认转发全部请求

- 支持拦截特定请求的 request 和 response

  - 不转发请求，直接返回自定义数据

  - 继续请求，修改入参或出参

- 缓存接口数据，支持离线调试

自定义请求处理文件：

```js
// request/order/getOrderInfo.js
export default async function getOrderInfo(data, { ctx, next }) {
  // 转发请求
  // return next();

  // 修改入参或出参
  // const response = await next({ data });
  // return response;

  // await new Promise(resolve => setTimeout(resolve, 2000));
  // 返回自定义数据
  const result = {
    code: 0,
    message: 'success',
    data: {
      orderId: '1234567890',
      orderStatus: '已完成',
      orderAmount: 100.00,
      orderTime: '2022-01-01 12:00:00',
    }
  }
  console.log('=============> 自定义请求处理 getOrderInfo', JSON.stringify(result));
  return result;
}
```

核心代码：

```js
// 自定义请求处理中间件
export default async function customHandlerMiddleware (ctx, next) {
  try {
    // 有缓存数据，直接返回
    const cacheData = await loadFromCache(ctx);
    if (cacheData) {
      handleResponse(ctx, cacheData);
      return;
    }

    // 根据请求路径，查找自定义处理文件
    // 例如：/order/getOrderInfo -> request/order/getOrderInfo.js
    const customRequestPath = path.join('request', this.ctx.url.split('?')[0] + '.js');
    const customRequest = fs.existsSync(customRequestPath) 
                            ? (await import(filePath)).default 
                            : null;

    if (customRequest && typeof customRequest === 'function') {
      // 使用自定义处理方法，不转发请求
      const data = { ...ctx.query, ...ctx.request.body };
      const response = await customRequest(data, { ctx, next });
      handleResponse(ctx, response);
      return;
    }

    // 转发请求
    const response = await next();
    handleResponse(ctx, response);
    await saveToCache(ctx, response);
  } catch (error) {
    handleError(ctx, error, '请求处理出错');
  }
}
```
