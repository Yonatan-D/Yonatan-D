# Gists

## 结束指定程序名的所有进程

```bash
ps -ef | grep 程序名 | awk '{print $2}' | xargs kill -9
```

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

## 加签和加密的区别

简单的说，两个区别： 应用场景不同：签名是为了宣告所有权，加密是为了秘密传送信息； 实现方式不同：签名是用私钥，公钥用来验证；加密使用公钥，解密用私钥。

A的签名只有A的公钥才能解签，这样B就能确认这个信息是A发来的；
A的加密只有B的私钥才能解密，这样A就能确认这份信息只能被B读取。