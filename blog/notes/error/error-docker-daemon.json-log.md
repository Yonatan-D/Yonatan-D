# docker 添加 log 配置后启动失败

报错信息:

```sh
：the following directives are specified both as a flag and in the configuration file
```

原因是 docker 启动的时候传入了命令行参数，同时也指定了配置文件，两个配置发生了冲突。

删一下 docker 启动文件里的配置即可。解决办法：

```sh
# 找启动文件 
vim /usr/lib/systemd/system/docker.service 
# 以及这个配置文件
vim /etc/sysconfig/docker
```

参考：https://www.cnblogs.com/cocowool/p/docker_daemon_log_driver.html