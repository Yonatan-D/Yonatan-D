# 报错解决

## redis 安装报错 jemalloc/jemalloc.h: No such file or directory

首先编译 Redis 需要先安装 gcc-c++

然后这个报错是因为没有清除上次编译残留文件

清理后重新编译就可以了：

```sh
make distclean && make
```

## npm install grpc 失败

报错信息：

```sh
Failed to load gRPC binary module because it was not installed for the current system Expected directory
```

使用 grpc_node_binary_host_mirror 参数设置国内镜像地址。解决办法：

```sh
npm install grpc --grpc_node_binary_host_mirror=https://npmmirror.com/mirrors/ --registry=https://registry.npmmirror.com
```

> grpc_node_binary_host_mirror 可以配置到 .npmrc 文件中，永久生效

## kill掉redis进程后重启失败

报错信息：

```sh
process is already running or crashed
```

尝试kill掉 /var/run/redis.pid 里的进程号，即 ps 查出来的第一行的 pid，结束进程。再次重启可能会出现 `process is already running or crashed` 的报错，这是因为进程号不一致导致的。

删掉 /var/run/redis.pid ，重启 redis 即可解决：

```sh
rm /var/run/redis.pid
```

## linux执行shell脚本报错：No such file or directory

windows下创建的编码格式是doc，在linux下执行会报 No such file or directory。解决方法是：用 vim，touch 命令新建文件；已经创建的用 :set ff 查看和转换，或者使用 notepad++ 等文本编辑器快速转换。

解决办法：

```sh
:set ff=unix
```

## LF will be replaced by CRLF

报错信息：

```sh
warning: LF will be replaced by CRLF
warning: LF will be replaced by CRLF in xxxxx
The file will have its original line endings in your working directory.
```

在 Windows 系统中，Git 将把 Unix 风格的换行符（LF）替换为 Windows 风格的换行符（CRLF）。

跨平台开发中常见问题，可以关闭自动替换避免警告。解决办法：

```sh
git config --global core.autocrlf false
```

## UOS 遇到 Error: ENOSPC: System limit for number of file watchers reached

报错信息：

```sh
Error: ENOSPC: System limit for number of file watchers reached
```

文件监视程序的系统产生了限制，达到了默认的上限，需要增加限额。解决办法：

```sh
echo fs.inotify.max_user_watches = 524288 | sudo tee -a /etc/sysctl.conf 
sudo sysctl -p
```

## GitLab 容器启动报错

报错信息：

```sh
/opt/gitlab/embedded/bin/runsvdir-start: line 24: ulimit: pending signals: cannot modify limit: Operation not permitted
/opt/gitlab/embedded/bin/runsvdir-start: line 37: /proc/sys/fs/file-max: Read-only file system
/opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/json-2.1.0/lib/json/common.rb:156:in `parse': 765: unexpected token at '' (JSON::ParserError)
	from /opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/json-2.1.0/lib/json/common.rb:156:in `parse'
	from /opt/gitlab/embedded/service/omnibus-ctl/check_config.rb:32:in `block in load_file'
	from /opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/omnibus-ctl-0.5.0/lib/omnibus-ctl.rb:193:in `block in add_command'
	from /opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/omnibus-ctl-0.5.0/lib/omnibus-ctl.rb:730:in `run'
	from /opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/omnibus-ctl-0.5.0/bin/omnibus-ctl:31:in `<top (required)>'
	from /opt/gitlab/embedded/bin/omnibus-ctl:23:in `load'
	from /opt/gitlab/embedded/bin/omnibus-ctl:23:in `<main>'
```

无法执行调整 file-max 的系统限制。解决办法：

```sh
# /etc/sysctl.conf 文件末尾追加 fs.file-max = 2097152
echo fs.file-max = 2097152 | sudo tee -a /etc/sysctl.conf

# 重载配置
docker exec -it gitlab gitlab-ctl reconfigure
```

## docker 添加 log 配置后启动失败

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

## yarn install 报错 is incompatible with this module

报错信息：

```sh
fsevents@1.2.13: The platform "win32" is incompatible with this module
# 或
@yarnpkg/core@2.4.0: The engine "node" is incompatible with this module. Expected version ">=10.19.0". Got "10.16.0"
```

报了个 ">=10.19.0". Got "10.16.0"。当前 node 版本 10.16.0 低了，需要升级到 10.19.0 以上版本（yarn 版本对 node 版本有要求）

如果不想升级 node 版本，可以选择忽略这个报错。解决办法：

```sh
yarn config set ignore-engines true
```
