# Gists

## 结束指定程序名的所有进程

```bash
ps -ef | grep 程序名 | awk '{print $2}' | xargs kill -9
```

## windows 结束指定程序名的所有进程

```cmd
taskkill /f /im 程序名.exe
```

## Github520 - fetch_github_hosts

```bat
powershell -Command "& { $remote='https://raw.hellogithub.com/hosts'; $hostsPath='%SystemRoot%\System32\drivers\etc\hosts'; $reader=[System.IO.StreamReader]::new($hostsPath, $true); $hostsContent=$reader.ReadToEnd(); $encoding=$reader.CurrentEncoding; $reader.Close(); $newContent=$encoding.GetString((Invoke-WebRequest -Uri $remote -UseBasicParsing).Content).TrimEnd(); $pattern='(?s)# GitHub520 Host Start.*?# Github520 Host End'; $newHostsContent=$hostsContent -replace $pattern, ''; if (-not $newHostsContent.EndsWith([Environment]::NewLine)) { $newHostsContent+=[Environment]::NewLine; } [System.IO.File]::WriteAllText($hostsPath, $newHostsContent + $newContent, $encoding); }"
```

> https://github.com/521xueweihan/GitHub520/issues/274

## 查看当前目录下所有文件大小并排序

```bash
du -h --max-depth=1 | sort -hr
```

## 查看执行过的命令带日期时间

```sh
history -i
```

## 查看 linux 设备 CPU 核数

```sh
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
```

## 按两下 Esc 键往上条命令或者当前正在输入的命令前加上（或删去） "sudo"

添加到 .bashrc 或 .zshrc 文件

```bash
sudo-command-line() {
  [[ -z $BUFFER ]] && zle up-history
  if [[ $BUFFER == sudo\ * ]]; then
    LBUFFER="${LBUFFER#sudo }"
  elif [[ $BUFFER == $EDITOR\ * ]]; then
    LBUFFER="${LBUFFER#$EDITOR }"
    LBUFFER="sudoedit $LBUFFER"
  elif [[ $BUFFER == sudoedit\ * ]]; then
    LBUFFER="${LBUFFER#sudoedit }"
    LBUFFER="$EDITOR $LBUFFER"
  else
    LBUFFER="sudo $LBUFFER"
  fi
}
zle -N sudo-command-line
bindkey "\e\e" sudo-command-line
```

## Wayland Fcitx 漏字修复

启动命令追加参数

```bash
--enable-features=UseOzonePlatform --ozone-platform=wayland --enable-wayland-ime
```

## nmap

```sh
# 获取主机版本信息
nmap -sV 172.16.0.1

# 获取主机操作系统信息
nmap -O 172.16.0.1
```

## 查看系统是32位还是64位

```bash
getconf LONG_BIT
```

## 查看 Windows 端口使用

```cmd
netstat -ano|findstr "10050"
```

## windows禁ping

```cmd
# 允许被ping
netsh firewall set icmpsetting 8
# 禁止被ping
netsh firewall set icmpsetting 8 disable
```

## windows 双网卡路由配置

目的：实现同时内外网访问

实现：

- 172.16.x.x -> 192.168.x.x

- 其他 -> 互联网

步骤：

```cmd
# 先移除默认策略
route delete 0.0.0.0

# 添加 172.16 段转发到内网网关
route add 172.16.0.0 mask 255.255.0.0 <内网网关> metric 20 -p

# 其它默认转发到外网网关
route add 0.0.0.0 mask 0.0.0.0 <外网网关> metric 22 -p

# 查看路由表
route print
```

## js 清空数组

看到一处注释，“**千万不要**在布局函数里面对routes进行重新赋值，如`routes=[]`，但是你可以使用`routes.length=0`来清空路由数组，然后重新添加路由对象到数组里，总之，不要改变routes对象的指针指向。”

引起了我的思考。以往也在别的文章看过 arr.length = 0 的操作，这个操作好吗？

有非常多的方法来清空一个已经存在的数组：

```js
var arr = [1,2,3,4];
```

**方法1： 直接赋值**

```js
arr = [];
```

将空数组直接赋值给变量 arr，新数组与原无引用关系。对新数组的操作不会影响原数组。性能最快，但改变了引用。

**方法2： 设置 length**

```js
arr.length = 0
```

直接将数组长度设置为0，速度次之。当"strict mode"严格模式时，因 arr.length 是只读的，此方法将不起作用。

**方法3： splice**

```js
arr.splice(0, arr.length)
```

这种方法会返回删除的所有元素，并形一个新的数组，保持对数组的引用。性能最慢。

也有其它诸如 pop 方法循环删除，其实看下来也明白了开发无银弹，要根据具体场景选择合适的方法。

## 快速删除node_modules

```sh
# linux
rm -rf node_modules

# windows
rd /s /q node_modules
```

## 添加 Typora 到右键

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\Typora]
"icon"="D:\\Program Files\\Typora\\Typora.exe"
@="Open With Typora"

[HKEY_CLASSES_ROOT\*\shell\Typora\command]
@="D:\\Program Files\\Typora\\Typora.exe" "%1"
```

## 删除运行输入过的命令

```bat
@ECHO OFF
REG DELETE HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU /f
echo "已清除历史命令"
PAUSE

rem .REG 的写法
rem Windows Registry Editor Version 5.00
rem [HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU] "MRUList"=-
```

## Vue组件重绘，唯一key的用法 

通过 `:key="key"` 实现，原理看[官方文档](https://cn.vuejs.org/api/built-in-special-attributes.html#key)。所以当 key 值变更时，会触发重新渲染。以前我的做法是通过watch监听数据变更。

## 摄像枪RTSP地址规则

> [海康摄像机、NVR、流媒体服务器、回放取流RTSP地址规则说明
](https://www.cnblogs.com/elesos/p/9881690.html)
> 
> [海康、大华网络摄像机RTSP URL格式组成及参数配置](https://zhuanlan.zhihu.com/p/53548980)

## 为什么 eggjs 启动命令要加 --workers

docker容器内跑eggjs工程启动报错：首先，启动命令不指定workers线程数的话，会默认创建和CPU核数相当的线程数，这是前提。又因为，docker容器本身的限制（额外提下，Docke容器使用主机CPU资源是不受限制的。作者说，Node&&Docker的BUG，os.cpus()无法识别在docker里正确的核数），所以就冲突了。容器化的egg最好直接指定worker进程数量（小一点的数，比如2），另外真实机器建议worker数和CPU核数一致。

https://github.com/eggjs/egg/issues/3088

## 可视化大屏的数据对接

> 来源：https://zhuanlan.zhihu.com/p/345634389

数据大屏接入数据的方式主要有：API接口、数据库、静态文件

API接口：在没有直接可复用接口时，不要偷懒，在前端同时对返回做大量数据的聚合操作，不仅会导致大屏响应变慢，而且也不容易保证数据的准确性。

数据库：数据可靠性和返回延时，查询sql的质量很重要，也会导致大屏响应缓慢甚至组件报错。

静态文件：在客户没有能力开发接口，也没有现有的数据库时，这时只能采用静态文件的形式作为大屏的数据来源。一般会采用csv格式（或者excel？）的文件，用户上传的每一个csv文件相当于数据库的一张表，需要根据大屏内容为每个csv文件设计合理的字段。

## 加签和加密的区别

简单的说，两个区别： 应用场景不同：签名是为了宣告所有权，加密是为了秘密传送信息； 实现方式不同：签名是用私钥，公钥用来验证；加密使用公钥，解密用私钥。

A的签名只有A的公钥才能解签，这样B就能确认这个信息是A发来的；
A的加密只有B的私钥才能解密，这样A就能确认这份信息只能被B读取。