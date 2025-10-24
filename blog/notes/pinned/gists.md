# Gists

## 结束指定程序名的所有进程

```bash
ps -ef | grep 程序名 | awk '{print $2}' | xargs kill -9
```

## 查看当前目录下所有文件大小并排序

```bash
du -h --max-depth=1 | sort -hr
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