## Zabbix 安装部署

server 端仅支持基于 Linux 的系统，官网有 Docker 安装方式

客户端支持 Linux、Windows

Docker步骤：https://www.cnblogs.com/lz1996/p/12625349.html


### 服务端安装

Network

```sh
docker network create -d bridge zabbix_net
```

mysql

```sh
docker run -dit -p 33306:3306 --name zabbix-mysql --network zabbix_net --restart always -v /zabbix/mysql/data:/var/lib/mysql -v /etc/localtime:/etc/localtime -e MYSQL_DATABASE="zabbix" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix@2021" -e MYSQL_ROOT_PASSWORD="root@2021" mysql:5.7
```

zabbix-server

```sh
docker run --name zabbix-server --network zabbix_net --restart always -e DB_SERVER_HOST="zabbix-mysql-server" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix@2021" -d zabbix/zabbix-server-mysql:centos-latest
```

zabbix-web

```sh
docker run --name some-zabbix-web-nginx-mysql -e DB_SERVER_HOST="zabbix-mysql-server" -e MYSQL_USER="zabbix" -e MYSQL_PASSWORD="zabbix@2021" -e ZBX_SERVER_HOST="zabbix-server" -e PHP_TZ="+08:00" -d zabbix/zabbix-web-nginx-mysql
```


### 客户端安装（windows）

zabbix-agents

安装包下载：https://cdn.zabbix.com/zabbix/binaries/stable/5.0/5.0.10/zabbix_agent-5.0.10-windows-amd64-openssl.msi

步骤：https://www.cnblogs.com/djlsunshine/p/10660085.html

zabbix_agentd.win.conf 

```sh
LogFile=c:\zabbix\zabbix_agentd.log

Server=192.168.2.80

ServerActive=192.168.2.80

Hostname=USERLEN-1KRKSMJ
```

agent安装与卸载

```bash
cmd /c "C:\Zabbix\bin\win64\zabbix_agentd.exe -c C:\Zabbix\conf\zabbix_agentd.win.conf -i"
cmd /c "C:\Zabbix\bin\win64\zabbix_agentd.exe -c C:\Zabbix\conf\zabbix_agentd.win.conf -s"

cmd /c "C:\Zabbix\bin\win64\zabbix_agentd.exe -c C:\Zabbix\conf\zabbix_agentd.win.conf -x"
cmd /c "C:\Zabbix\bin\win64\zabbix_agentd.exe -c C:\Zabbix\conf\zabbix_agentd.win.conf -d"

-c ：指定配置文件所有位置
-i ：安装客户端
-s ：启动客户端
-x ：停止客户端
-d ：卸载客户端
```

**查看 Windows 端口使用**

C:\Users\Administrator>netstat -ano|findstr "10050"

```sh
C:\Users\Administrator>netstat -ano|findstr "10050"
  TCP    0.0.0.0:10050          0.0.0.0:0              LISTENING       8980
  TCP    [::]:10050             [::]:0                 LISTENING       8980
```

## zabbix-agent: 被监控接入

左侧导航 ：配置 > 主机，可以直接选择已有监控列表里对应的 linux 或 windows 服务器，全克隆，修改成要监控的服务器的主机名称、客户端、端口

服务器也要配置，这里使用 ZBX 方式最简单

`linux`：直接下载 

```bash
wget https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-agent-5.0.9-1.el7.x86_64.rpm
rpm -ivh zabbix-agent-5.0.9-1.el7.x86_64.rpm
vim /etc/zabbix/zabbix_agentd.conf
# Server, ServerActive 设置为 zabbix 服务器地址(186)
# ListenPort 默认是 10050，有占用的话可以改成其他
# 有个很迷惑的地方，端口占用 zabbix agentd 没起来，报的错却是另一台正常监控的服务器连不上：
# zabbix: Get value from agent failed: cannot connect to [[172.17.10.188]:10050]: Connection refused
#
# 启动，重载配置
zabbix_agentd -c zabbix_agentd.conf 
```

`windows`：需要安装

zabbix_agent-5.0.15-windows-amd64-openssl.msi

安装引导页，按照提示设置成本机的 IP 和默认 10050 端口
