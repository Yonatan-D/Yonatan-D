# UOS 遇到 Error: ENOSPC: System limit for number of file watchers reached

报错信息：

```sh
Error: ENOSPC: System limit for number of file watchers reached
```

文件监视程序的系统产生了限制，达到了默认的上限，需要增加限额。解决办法：

```sh
echo fs.inotify.max_user_watches = 524288 | sudo tee -a /etc/sysctl.conf 
sudo sysctl -p
```