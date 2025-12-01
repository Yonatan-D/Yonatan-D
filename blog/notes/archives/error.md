# 报错解决

## Vue前端项目较大，运行和编译都报错内存溢出

```
报错如下：
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
```

【已弃用】使用这个插件，配置后运行 fix-memory-limit 解决：[increase-memory-limit](https://www.npmjs.com/package/increase-memory-limit)

从 2017 年 8 月发布的 Node.js v8.0 开始，可以设置全局环境变量：

```
export NODE_OPTIONS=--max_old_space_size=4096
```

推荐只修改对应项目的 package.json 的 scripts 命令：

```
--max-old-space-size=4096
```

vue-cli2 按上面方法可行，vue-cli3 则需要这么写：

```bash
node --max_old_space_size=4096 node_modules/.bin/vue-cli-service build --mode test
```

##  Egg开启自动生成dts报错：Symbol.for('egg#eggPath') is required on Application

背景：Egg开启智能提示（https://zhuanlan.zhihu.com/p/56780733）。业务项目 yarn link 封装的框架项目，并在 package.json 添加 "egg": { "declarations": true }，启动应用 `npm run dev` 报错

参考：https://github.com/eggjs/egg/issues/3591

临时解决：

找到 project/node_modules/egg-core/lib/loader/egg_loader.js ，改成：

```js
proto === Object.prototype || proto === EggCore.prototype || proto.constructor.name === 'EggCore'
```

## colors.js被恶意提交导致控制台输出乱码

排查项目中是否有使用 colors 库

```sh
yarn why colors 
```
在 package.json 配置 resolutions 锁定正常版本

```json
"resolutions": {
  "colors": "1.4.0"
},
```

> 如果项目用的是 npm：运行 npm ls colors，配置 overrides（支持嵌套、重写特定模块下的子依赖）

## vsphere client 虚拟机 unknown 不可访问

背景：公司断电导致的7个服务器变为unknown，挂掉了一个磁盘。后来查出来是机房的一台光纤交换机没恢复供电，恢复后重启自动挂载启动解决。

马克一下找到的恢复方法，以防真的极端情况下需要恢复数据：[EXSI 5.5 虚拟机，使用*-flat.vmdk恢复的方法](https://blog.51cto.com/unclemao/1729428)

## 人大金仓(kingbase)数据库把空字符串当成Null

代码里有段sql是 `xxx字段<>''`，意思是查询该字段不为空字符串的记录，但是人大金仓数据库把空字符串当成Null，所以查询结果为空。

复现步骤：

```sql
SELECT '' = '';
```

输出：false

解决只需修改人大金仓数据库配置：

```toml
ora_input_emptystr_isnull = false
```

## 记一次误操作卸载 iptabels-server 后如何恢复 docker 服务

把同时被删掉的15个包（service等命令），都安装回来

```bash
yum -y install iptables abrt-addon-vmcore abrt-cli abrt-console-notification dhclient dracut-network firewalld initscripts iproute kbd kexec-tools libstoragemgmt libstoragemgmt-python libstoragemgmt-python-clibs plymouth plymouth-scripts
```

根据系统日志逐一排查问题

```sh
tail -f /var/log/messages
```

原来的/etc/init.d/下的启动脚本全都丢失。恢复下 /etc/systemd/system/ 目录下的 docker.service 和 docker.socket 文件：

```bash
#docker.service中添加内容
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID

LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

TimeoutStartSec=0

Delegate=yes

KillMode=process

Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s

[Install]
WantedBy=multi-user.target
```

```bash
#docker.socket中添加内容
[Unit]
Description=Docker Socket for the API
PartOf=docker.service

[Socket]
ListenStream=/var/run/docker.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.targe
```

```bash
#添加文件权限
chmod +x /etc/systemd/system/docker.service
# 先重启 systemctl 守护进程
sudo systemctl daemon-reload
```

补上缺失的网桥（卸载时连同被清理了）：

```bash
ip link add name docker0 type bridge
ip addr add dev docker0 172.17.0.1/16
```

可以启动了：

```bash
sudo systemctl start docker
# 设置自启动
sudo systemctl enable docker
```

## chrome 升级版本后 iframe 嵌套的第三方页面白屏

问题分析：

Google 在2020年2月4号发布的 Chrome 80 版本默认屏蔽所有第三方 Cookies 跨域携带，使用了 SameSite Cookie 的策略保护跨站请求伪造(CSRF)和用户跟踪，所以 iframe 中 Cookies 无法传递。

应急处理：

谷歌浏览器访问 chrome://flags/
搜索 SameSite by default cookies，参数设置成disabled

【没试验过】https://blog.csdn.net/lj1530562965/article/details/117034307

```js
import Cookies from 'js-cookie'

export function setToken (val) {
   if (window.location.protocol === "https:") {
    return Cookies.set("scs_token", val, {sameSite: "none", secure: true});
   }
   return Cookies.set("scs_token", val);
}
```

?> 代码解决方案：在请求头携带 Cookies

## eggjs 站点通过 nginx 无法访问，而直连可以访问

该服务在 nginx 上做了负载均衡，单独访问每个节点都可以访问，但通过 nginx 访问时，偶发性无法访问。

处理请求业务是由 egg.js 的 agent 进行分发，随机发送给其中一个 worker ，而有一个 worker 进程陷入死循环，如果分配给它，请求并不会执行，会处于等待状态，直到超时，也就显示 nginx 的超时错误。没有分配给有问题的 worker 的话，能正常执行请求业务。

————————————————

为什么站点日志没有错误日志输出？

该错误是通过 `console.error()` 进行输出的，可能是日志级别配置导致不存在日志文件里面。因为是用 docker 部署的，docker 日志捕获到了这次错误

————————————————

为什么内存会缓慢下降

这是因为进程陷入死循环，在不停的执行，没有得到资源释放，在V8的垃圾回收机制中没有被回收。并且分配给这个 `worker` 的请求，会因为阻塞而无法得到释放，就会慢慢的消耗内存，只有重启才能恢复正常。

## Docker和Firewalld冲突

内网有两个网段 172.17 和 192.168，网络防火墙已允许 172.17.10.90 主机被 192.168.0.130 主机访问，但如今 ping 不通，telnet 也不通，原因是 Docker 网段和内网主机网段冲突了。解决方法：[`bip : 设置 docker 网桥的 IP 地址段，默认是 172.17.0.1/16。避免和内网服务器 IP 地址冲突`](/notes/tools/docker/docker-daemon)

当时的踩坑记录：

首先想的是设置主机防火墙。放行 172.17.10.90

```bash
# 临时的，重启会失效
route add -net 172.17.10.90 netmask 255.255.255.255 gw 192.168.0.1
# 永久生效
vim /etc/sysconfig/static-routes
any net 172.17.10.90 netmask 255.255.255.255 gw 192.168.0.1
```

此时已经能 ping 通，继续放行端口

```bash
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --list-ports
firewall-cmd --reload
```

能通了，但过会又不行了，时灵时不灵。重启防火墙只能维持正常一段时间

```bash
systemctl restart firewall
```

这时候猜到了可能又是 docker 在捣鬼了，docker 的端口映射也是基于 iptables，我用 iptables 开的端口，给冲突了，看一下防火墙日志：

```bash
systemctl status firewalld
```

果然，docker 出现在了日志。这也能解释上面我在执行 `firewall-cmd --list-ports` 时消失了几个原有的端口。

关闭 130 主机上的防火墙，问题解决

```bash
systemctl stop firewalld
systemctl disable firewalld
```

~~将 firewalld 换成 iptables~~ 

```sh
-A INPUT -p tcp -m tcp --dport 3306 -j ACCEPT
```

后记：这是 Docker 网段和内网主机网段冲突导致的，应该配置 bip 设置别的网段。

## 项目报错找不到name，原因是浏览器版本低

浏览器不支持 `Import maps` ：这个提案允许控制 js 的 `import` 语句或者 `import()` 表达式获取的库的url，并允许在非导入上下文中重用这个映射。

浏览器支持情况：https://caniuse.com/?search=importMap

```js
import moment from "moment";
import { partition } from "lodash";

<script type="importmap">
{
  "imports": {
    "moment": "/node_modules/moment/src/moment.js",
    "lodash": "/node_modules/lodash-es/lodash.js"
  }
}
</script>

// 解析成：
import moment from "/node_modules/moment/src/moment.js";
import { partition } from "/node_modules/lodash-es/lodash.js";
```

> 相关阅读：  
> [从systemjs的使用学习js模块化](https://segmentfault.com/a/1190000022278429)

## supervisorctl start app 遇到错误：ERROR (spawn error)

```sh
# 使用命令查看错误信息：
supervisorctl tail program_name stderr

# 错误原因是：
# /etc/supervisord.conf
[program:app]
command=node ~/yd-supervisord/app.js  #错误：不能用 ~ 字符
command=node /yd-supervisord/app.js   #正确
```

## CSS布局遇到：start value has mixed support, consider using flex-start instead

flex 属性写法不规范。start 值在不同浏览器中的支持情况不一致，使用 flex-start 在所有支持 Flexbox 的浏览器中都能正常工作

```css
.justify-start {
   display: flex;
   justify-content: flex-start; /* 替代 start */
}
```

## history模式，部署后地址栏访问无效，刷新后404

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

## sequelize 连接 mssql 报错：ERROR: Server requires encryption, set 'encrypt' config option to true.

原因：SQL Azure与SQL Server有区别。sequelize的mssql支持连接azure（因为azure本身就是基于sql server的），但是要加上这个配置：

https://github.com/sequelize/sequelize/issues/3240

解决方法：https://stackoverflow.com/questions/56113296/set-encrypt-to-true-on-sequelize-cli-migrate-for-mssql

```js
module.exports = {
  development: {
    username: "USER",
    password: "PASSWORD",
    database: "DB_NAME",
    host: "HOST.net",
    dialect: 'mssql',
    dialectOptions: {  // 加上这段
      options: {
        encrypt: true
      }
    }
  } 
};
```

## wss（white shark system）白屏解决

参考：https://blog.csdn.net/xiang1009/article/details/102784625

1.安装64位wamp，页面空白，选择PHP->version->5.x版本

2.局域网访问，入站规则添加80端口，修改apache的httpd-vhosts.conf，将Require local改为Require all granted。

3.单击wamp运行图标，选择MySQL的my.ini，修改其中的sql-mode

将STRICT_ALL_TABLES改为STRICT_TRANS_TABLES

将NO_ZERO_DATE和NO_ZERO_IN_DATE删除

保存后，重启即可解决

## linux 开机错误 Entering emergency mode. Exit the shell to continue.

https://blog.csdn.net/whatday/article/details/104819743

输入 journalctl 可以查看系统的日志信息

解决方法：

```sh
xfs_repair -v -L /dev/dm-0

-L 选项指定强制日志清零，强制xfs_repair将日志归零，即使它包含脏数据（元数据更改）。
```

## nginx 因 referer 头导致 403

a标签：referrerpolicy="no-referrer" （A标签页面跳转是403，回车后正常问题）

nginx：valid_referers none blocked 域名或IP; （添加白名单）

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
