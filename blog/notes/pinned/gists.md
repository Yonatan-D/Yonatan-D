# Gists

## 结束指定程序名的所有进程

```bash
# linux
ps -ef | grep 程序名 | awk '{print $2}' | xargs kill -9

# windows
taskkill /f /im 程序名.exe
```

## 根据进程号PID查找启动程序的全路径

```bash
ps -ef | grep xxx

# 假如PID为 1234
ls -l /proc/1234/cwd
```

```bash
sc delete [服务名称]
```

## Github520 - fetch_github_hosts

windows

```bat
powershell -Command "& { $remote='https://raw.hellogithub.com/hosts'; $hostsPath='%SystemRoot%\System32\drivers\etc\hosts'; $reader=[System.IO.StreamReader]::new($hostsPath, $true); $hostsContent=$reader.ReadToEnd(); $encoding=$reader.CurrentEncoding; $reader.Close(); $newContent=$encoding.GetString((Invoke-WebRequest -Uri $remote -UseBasicParsing).Content).TrimEnd(); $pattern='(?s)# GitHub520 Host Start.*?# Github520 Host End'; $newHostsContent=$hostsContent -replace $pattern, ''; if (-not $newHostsContent.EndsWith([Environment]::NewLine)) { $newHostsContent+=[Environment]::NewLine; } [System.IO.File]::WriteAllText($hostsPath, $newHostsContent + $newContent, $encoding); }"
```
> https://github.com/521xueweihan/GitHub520/issues/274

linux

```bash
sudo sh -c 'sed -i "/# GitHub520 Host Start/Q" /etc/hosts && curl https://raw.hellogithub.com/hosts >> /etc/hosts'
```

## 查看当前目录下所有文件大小并排序

```bash
du -h --max-depth=1 | sort -hr
```

## 查找"/"目录下所有大于100M的所有文件

```bash
find / -type f -size +100M -print0 | xargs -0 du -h | sort -nr
```

## 磁盘满了查不到占用

```bash
# 查
lsof | grep delete

# 查杀。需谨慎，建议用上面命令查完手动杀
lsof | grep delete| awk '{print $2} '|xargs kil -9
```

如果是 nohup.out 的占用, 使用上述的查杀命令会把进程给结束, 生产环境就不能这么操作了, 建议查完手动杀。

使用 nohup 命令启动的程序会在当前目录下自动生成 nohup.out 日志文件，遇到过一次启动 .net 程序，dll 进程产生的日志很恐怖，如果不需要这个日志文件那么就在启动命令加上 `> /dev/null` 丢弃掉日志。

不停止服务的做法是，手动将 /dev/null 覆盖到 nohup.out 起到清空作用：

```bash
cp /dev/null nohup.out
```

想继续保留这个日志文件，可以编写一个定时任务执行上述命令，已达到减少体积。

或者保留最近的日志, 

```bash
log=`tail -n 10000 nohup.out`
echo "$log" > nohup.out
```

## 查看进程的占用连接数

成功连接数：

```bash
netstat -na|grep -i "8880"|grep ESTABLISHED|wc -l
```

失败连接数（超时）：

```bash
netstat -na|grep -i "8880"|grep TIME_WAIT|wc -l
```

## 查看执行过的命令带日期时间

```sh
history -i
```

## 查看 linux 设备 CPU 核数

```sh
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l
```

## 临时设置代理

linux:

```sh
export HTTP_PROXY="http://ip:port"
export HTTPS_PROXY="https://ip:port"
```

windows:

```sh
# 临时设置代理
set http_proxy=http://ip:port
set https_proxy=https://ip:port
```

## Linux 注销时执行脚本，删除 history 记录

每个用户登出时都清除输入的命令历史记录，可以在`.bash_logout` 文件中添加下面这行`rm -f $HOME/.bash_history` 。这样，当用户每次注销时，`.bash_history `文件都会被删除.【需谨慎，使用rm删除命令时加上-i参数让用户确认】

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

## windows 删除服务

Win+R 输入 services.msc 打开服务，选中列表项右键属性查看服务名称，cmd执行以下命令：

## windows输入法不见了

Win+R 运行 ctfmon.exe

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

## powershell脚本启动网卡

```ps1
# 检查是否已经以管理员权限运行
$isAdmin = ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")

# 如果不是管理员权限，则尝试以管理员权限运行
if (-not $isAdmin) {
    Start-Process powershell.exe "-NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File `"$PSCommandPath`"" -Verb RunAs -Wait
    Exit
}

# 在这里添加你的命令
netsh interface set interface "网卡名称" enable # 开启网卡
# netsh interface set interface "网卡名称" disabled # 关闭网卡
```

## 快速启动本地服务

有2个工具：http-server，anywhere

```sh
npx http-server -p 3000 

npx anywhere -p 3000
```

## 快速删除node_modules

```sh
# linux
rm -rf node_modules

# windows
rd /s /q node_modules
```

## 自动监测进程的脚本

!> 专业的事交给专业的人做，脚本仅供参考。推荐工具：Systemd、Supervisor、PM2

```bash
#!/bin/bash

#######################################################
# 镜像 root 目录新增如下:
#
# /root
#
#   |- daemon.sh
#
#   |- cleanup_log.sh
#
# 使用：
# crontab -e
# 10 * * * * docker exec myweb bash -c "cd /root && ./#aemon.sh [参数: 需要被守护的服务名]"
# 0 0 * * * docker exec skylandweb bash -c "cd /root && #leanup_log.sh"
#
# 第一条为每10分钟去执行服务守护脚本
# 第二条为每天0时清除日志文件
#######################################################

serverName=$1
serverLog=/root/logs/$1.log
time=$(date +"%F %H:%M:%S")
serverPID=$(ps -ef | grep $serverName | grep -v 'grep' | awk '{print $2}')

Daemon () {
  if[ serverPID ];then
    echo "[$serverPID] [$time] server working"
  else
    kill -9 $serverPID
    echo "[$serverPID] [$time] server stoped"
    echo "[error] server need to restart"
    echo "[info]  waiting..."
    # 启动命令
    #
    #
  fi
  echo "--------------------------"
  sleep 1
  done
}

Daemon>>$serverLog
```

## 移除 Windows DNS 缓存

telnet 网关 (如: telnet 192.168.14.254)

display current-configuration (看到所有段的网关信息)

system-view (进入视图，我猜测是当前网关的ip池)

display this (显示当前段的网关信息，即: 192.168.14.254)

undo dns-list x.x.x.x (移除DNS)

qu (退出)

save (保存，网关会重启，短暂断网)

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

## 内网发送短信

遇到一个内网环境下发送短信的需求，搜到一些实现思路：

1. 使用内网数据库，外网程序轮询该数据库，读取短信内容，调用短信接口发送【需要一台能被外网访问的内网服务器，单向网络。如果外网服务器不能主动访问内网服务器，看第5点摆渡的做法】

2. 内网直接有专线连接到运营商的短信系统，用内网地址就可以调用运营商短信接口【需要运营商专线，成本较高】

3. 使用“短信猫”外接设备。内网终端生成待发送短信，通过短信猫直接发送【需要内网部署短信猫设备】

4. 用一台能访问内外网的服务器，接收短信并上传到短信服务平台，平台解析后调用短信接口发送【需要一台内外网策略通的服务器作为中转】

5. 信息摆渡：[政务内外网环境，内网单向访问外网【内网穿透】数据传输解决方案](https://my.oschina.net/thinkgem/blog/4624519)

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