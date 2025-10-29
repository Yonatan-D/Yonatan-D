# Supervisor使用

配置文件 /etc/supervisord.conf

默认web管理地址 http://127.0.0.1:9001

子进程配置文件目录 /etc/supervisord.d/

使用：

```bash
# 安装启动
yum install supervisor
# apt-get update
# apt-get install supervisor

systemctl start supervisord.service
systemctl enable supervisord.service
# /etc/init.d/supervisor start
# /etc/init.d/supervisor stop

# 子进程示例
[program:test] 
directory=/opt/bin    #脚本目录
command=/opt/bin/test #脚本执行命令
autostart=true 
autorestart=false 
stderr_logfile=/tmp/test_stderr.log 
stdout_logfile=/tmp/test_stdout.log 
#user = test  
```

web查看进程：

```toml
[inet_http_server]
port=0.0.0.0:9001   ;IP按需配置     
username=user              
password=123
```

基本命令：

```bash
supervisorctl status        //查看所有进程的状态
supervisorctl stop app      //停止app,也可以all
supervisorctl start app     //启动app,也可以all
supervisorctl restart app   //重启app,也可以all
supervisorctl update        //配置文件修改后使用该命令加载新的配置
supervisorctl reload        //重新启动配置中的所有程序
```

> 参考教程：https://www.cnblogs.com/zhoujinyi/p/6073705.html