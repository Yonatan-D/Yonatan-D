# Gits

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