# Docker

## 安装

1. 自动化脚本安装

```bash
curl -sSL https://get.docker.com | sh
```

2. 逐步安装

Docker官网下载：https://download.docker.com/linux/static/stable/

```bash
cd ~
wget https://download.docker.com/linux/static/stable/x86_64/docker-18.06.3-ce.tgz
tar zxvf docker-18.06.3-ce-x86_64.tgz
cp docker/* /usr/bin/


vim /etc/systemd/system/docker.service
# -------------------------------- #
# docker.service start
# -------------------------------- #
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
# -------------------------------- #
# docker.service end
# -------------------------------- #



vim /etc/systemd/system/docker.socket
# -------------------------------- #
# docker.socket start
# -------------------------------- #
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
# -------------------------------- #
# docker.socket end
# -------------------------------- #



chmod +x /etc/systemd/system/docker.service
systemctl daemon-reload
systemctl start docker
systemctl enable docker.service
```



### 常用命令

Docker 镜像和容器的常用操作命令

#### 查看镜像

```bash
docker images
```

#### 拉取镜像

```bash
docker pull 镜像地址
# 官方源有时候速度慢，可以使用国内镜像源
```

#### 制作镜像

Docker 镜像制作主要有两种方式 **Dockerfile** 和 **快速制作方式**。接下来以实例演示如何通过这两种方式制作镜像。

##### 1. Dockerfile 制作镜像

制作一个内置 Git 环境的 ubuntu 镜像

创建 Dockerfile 文件，内容如下：

```dockerfile
FROM ubuntu:18.04
# 输入你的姓名和邮箱
MAINTAINER your_name <your_email>
# 安装Git
RUN apt-get install -y git
# 启动时运行这个命令
CMD ["/bin/bash"]
```

运行以下命令，build 镜像：

```bash
docker build -t myubuntu ./  #正式build, 命名为 myubuntu
```

build 完成后可以查看和使用镜像



##### 2. 快速制作镜像

将已有容器创建为镜像

查看容器，找到对应的 CONTAINER ID ， 例如： 15adabd78f9b

```bash
docker commit 15adabd78f9b myubuntu
```



#### 删除镜像

```bash
docker rmi 镜像名称或ID
```

#### 导入导出镜像

使用 save 和 load 命令

```bash
# 导出镜像
docker save myubuntu:latest > myubuntu.tar
# 导入镜像
docker load < myubuntu.tar
```



#### 创建容器

```bash
docker run -dt --name=容器名称 --restart=always -p 主机(宿主)端口:容器端口 镜像名称:版本号
```

-v 宿主机目录:容器目录

* 如果host机器上的目录不存在，docker会自动创建该目录
* 如果container中的目录不存在，docker会自动创建该目录
* 如果container中的目录已经有内容，那么docker会使用host上的目录将其覆盖掉

使用宿主机当前目录

```bash
# Powershell下使用：
-v ${pwd}:/www

# Linux下使用：
-v $(pwd):/www
```

#### 进入容器

```bash
docker exec -it 容器名称 /bin/bash
```

#### 查看全部容器

```bash
docker ps -a
```

#### 查看正在运行容器

```bash
docker ps
```

#### 删除容器

```bash
docker rm 容器名称或ID
```

#### 导入导出容器

使用 export 和 import 命令

```bash
# 导出镜像
docker export 15adabd78f9b > ubuntu-15adabd78f9b.tar
# 导入镜像
docker import ubuntu-15adabd78f9b.tar
```

export 不会保留镜像的层级信息，所以大小会比 save 小

#### 日志排查

```bash
docker logs xxx --tail 500

docker inspect xxx
tail -n 100 /var/lib/docker/containers/xxx/xxx-json.log
```

#### 进程排查

```bash
docker stats --no-stream | sort -rn -k 3
docker exec -it xxx ps -aux
```

#### 查询cpu和内存占用

```sh
docker stats --no-stream | sort -rn -k 3
```

--format格式化成表格或JSON

```sh
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker stats --no-stream --format "{\"name\":\"{{.Name}}\",\"cpu\":\"{{.CPUPerc}}\",\"mem\":\"{{.MemUsage}}\"}"
```

## 推荐配置

1. 打开 daemon.json 文件，没有就新建

```bash
vim /etc/docker/daemon.json
```

2. 添加以下内容

```json
{
  "registry-mirrors": ["https://registry.docker-cn.com"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "bip": "172.18.0.1/16"
  // "graph": "/home/docker" // 19.x 版本后已废弃，改为 "data-root"
}
```

谨慎修改 bip 和 graph 配置，否则可能会导致 docker 服务无法启动。注意 json 文件格式，最后一行不要加多余的逗号！

daemon.json 文件配置说明：

- `registry-mirrors` : 配置镜像源来加速镜像拉取。[DockerHub国内镜像加速源列表](https://github.com/dongyubin/DockerHub)

- `log-driver` : 设置 docker 日志驱动为 json-file

- `log-opts` : 设置 docker 日志文件大小为 10m，文件个数为 3 个。限制日志文件大小和个数，避免日志文件过大导致磁盘空间不足，设置后只对新添加的容器有效

- `bip` : 设置 docker 网桥的 IP 地址段，默认是 172.17.0.1/16。避免和内网服务器 IP 地址冲突

- ~~`graph`~~  `data-root` : 设置 docker 数据文件存放位置，默认是 /var/lib/docker。建议安装 docker 后马上设置

3. 重启 docker 服务

```bash
systemctl daemon-reload
systemctl restart docker
```

4. 验证配置是否成功

```bash
docker info | grep "Docker Root Dir"
```

## 白名单 (iptables)

拒绝所有访问

```bash
iptables -I DOCKER-USER -i ens33 -j DROP
```

拒绝对某容器的访问

```bash
iptables -I DOCKER-USER -i ens33 -d 172.17.0.2 -j DROP
```

放行IP对某容器的访问

```bash
iptables -I DOCKER-USER -i ens33 -s 192.168.150.220 -j ACCEPT
```

容器设置固定 IP (容器默认 172.17.x.x 每次重启是可能改变的，它是根据容器启动顺序分配的)

```bash
docker network create --subnet=xxx.xxx.xxx.0/24 newNetWork
docker run -itd --name dockerName --network=newNewWork --ip xxx.xxx.xxx.1 imageName
```

示例：

搭建一个 svn 服务，设置 IP 白名单

```bash
docker network create --subnet=172.18.0.0/24 svn-network
docker run --restart always --name svn -d  --network=svn-network --ip 172.18.0.101 -v /root/svn:/var/opt/svn -p 8081:3690 garethflowers/svn-server

# 拒绝所有IP对该容器的访问
iptables -I DOCKER-USER -i ens33 -d 172.18.0.101 -j DROP
# 放行特定IP对该容器的3690端口的访问
iptables -I DOCKER-USER -i ens33 -s 192.168.150.220 -p tcp -d 172.18.0.101 --dport 3690 -j ACCEPT

# 保存配置
# iptables-save > /etc/sysconfig/iptables
# 或者, 安装了 iptables-services 可以用
service iptables save

# 加载配置
# iptables-restore < /etc/sysconfig/iptables
```

## 清理占用的磁盘空间

### 1. docker system 命令

`docker system df` 类似于 Linux 上的 df 命令，用于查看 docker 磁盘空间占用情况

```
TYPE           TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images         147       36        7.204GB   3.887GB (53%)
Containers     37        10        104.8MB   102.6MB (97%)
Local Volumes  3         3         1.421GB   0B (0%)
Build Cache    0         0         0B        0B
```

删除关闭的容器、无用的数据卷和网络，以及 dangling 镜像(即无 tag 的镜像)

```bash
docker system prune
```

执行命令后输出如下：

```bash
WARING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - all dangling build cache

Are you sure you want to continue? [y/N] 
```

加了 -a 参数，可以将没有容器使用的镜像也一并删除

```bash
docker volume prune -a
```

### 2. 手动清理

(1) 删除所有关闭的容器：

```bash
docker ps -a | grep Exit | cut -d ' ' -f1 | xargs docker rm
```

(2) 删除所有未打标签的镜像：

```bash
docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
```

(3) 删除所有无名 `<none>` 镜像

```bash
docker rmi -f $(docker images -f dangling="true" -q)
```

(4) 删除所有同名镜像

```bash
docker rmi -f $(docker images | grep 镜像名称)
```

(5) 删除所有无用数据卷

```bash
docker volume rm $(docker volume ls -qf dangling=true)
```

### 3. 排查占用大的容器

默认 docker 数据都保存在 `/var/lib/docker` 目录下，假如修改了目录可以通过下面这条命令查询：

```bash
docker info | grep "Docker Root Dir"
```

进入 `/var/lib/docker` 目录，使用 du 命令查看占用空间最大的目录

```bash
[root@localhost docker]# du . -h --max-depth=1
84K ./buildkit
355M ./containers
24M ./image
100K ./network
14G ./overlay2
0   ./plugins
0   ./swarm
0   ./trust
735M   ./volumes
0   ./tmp
0   ./runtimes
15G .
```

可以看到占用空间最大的目录是 `/var/lib/docker/overlay2`，进入该目录，继续执行 du 命令，直到定位到具体的容器

一般有 2 种情况，一种是占用较大的是 containers 目录，另一种是 overlay2 目录，下面分别介绍如何排查占用大的容器

containers 目录：通过 docker ps -a 命令查看容器 ID，找到前缀一致的目录名称

overlay2 目录：因为 overlay2 目录下的文件名并不是容器 ID，所以需要使用下面的命令进行查询，再去找一致的容器 ID

```bash
docker ps -q | xargs docker inspect --format '{{.State.Pid}}, {{.Id}}, {{.Name}}, {{.GraphDriver.Data.WorkDir}}' | grep 目录名称
```

通常占用大的原因是容器内的应用产生了大量日志，解决方法是找到产生日志的源头做策略，例如：nginx 限制日志大小，.Net 程序等 nohup 启动时默认产生的 nohup.out 日志文件需要 `> /dev/null` 给丢弃掉...或者 docker 配置限制日志文件大小和数量，但这只对后面添加的容器有效

## 常见问题

### 迁移 docker 目录

docker 默认安装在 /var/lib/docker 目录下，假如一开始没有选择安装在最大的磁盘，后期可能会因为容器体积和日志文件占满磁盘，导致服务不可用。所以最好在安装完启动前就设置好 graph 属性，避免后期迁移。

迁移步骤如下：

1. 停止 docker 服务

```bash
systemctl stop docker
```

2. 备份原目录

```bash
mv /var/lib/docker /var/lib/docker.bak
```

3. 创建新目录

```bash
mkdir -p /home/docker
```

4. 迁移文件

```bash
cp -r /var/lib/docker/* /home/docker/
# 或者
# rsync -avz /var/lib/docker/ /home/docker/
```

5. 修改 docker 配置，没有就新建

```bash
vim /etc/docker/daemon.json

# 添加下面内容
{
  "graph": "/home/docker"
}
```

!> 19.x 版本后弃用 `graph` 属性，使用 `data-root` 属性

6. 重载 docker 服务

```bash
systemctl daemon-reload
systemctl start docker
```

7. 验证

```bash
docker info | grep 'Docker Root Dir'
```

成功修改的话将输出：Docker Root Dir: /home/docker

启动服务后确认下原先的镜像和容器是否正常，确定没问题后可以删除原目录



### 已运行的容器追加端口号

要给已运行的容器追加端口号，需要先停止容器，然后修改容器配置文件，最后重启容器

例如要给名为 myubuntu 的容器（ID 为 15adabd78f9b）追加 80 端口，操作步骤如下：

1. 停止容器

```bash
docker stop myubuntu
```

2. 修改配置文件

进入 containers 目录，用 ls 命令找到符合容器 ID 前缀的文件夹

```
cd /var/lib/docker/containers/15adabd78f9b...(省略)
```

找到 hostconfig.json 文件，在 PortBindings 里添加：

```json
"80/tcp": [{"HostIP": "", "HostPort": "80"}]
```

找到 config.v2.json 文件，在 ExposedPorts 里添加：

```json
"80/tcp": {}
```

3. 重启服务

```bash
systemctl restart docker
```



### 容器内时间与宿主机时间不同步

容器内使用 date 查看时间发现与宿主机时间不一致，可能会导致项目运行出错。此时如果使用 ntpdate 工具进行同步会出现报错，报错显示如下：

```bash
ntpdate: Can't adjust the time of day: Operation not permitted
```

解决方法：

1. 移除容器内的 localtime 文件

```bash
docker exec -it 容器ID mv /etc/localtime /root/
```

2. 执行以下命令，复制宿主机上的 localtime 文件到容器内

```bash
docker cp /root/localtime 容器ID:/etc/
```

3. 验证

```bash
docker exec -it 容器ID date
```

### 限制docker生成core文件

问题排查：服务器磁盘空间不足，排查发现是docker容器生成的core文件占用了大量空间

```bash
[root /]# ulimit -a
core file size          (blocks, -c) 0		 
# 0不启用, unlited不限制  单位kb

方法一: 修改 docker.service 的 ExecStart 这一行，追加 –default-ulimit core=0:0 # 禁用容器生成Core文件
方法一: docker run --ulimit core=0
```

### docker运行卷映射，宿主机没文件，容器内文件消失

背景：老大做了个 cnpmjs 镜像，我拿到服务器上部署时有个目录做卷映射不成功 (config不可以，download可以)

原因 (网上找到相似经历)：https://www.jianshu.com/p/530d00f97cbf

### docker应该使用 pm2-runtime

pm2-runtime 是为 Docker 容器设计的，它将应用程序置于前台，从而使容器保持运行状态

简单来说，容器的生命周期就是 `CMD` 或 `entrypoint` 的生命周期, 在使用 `CMD [ "pm2", "start","/app/server.js"]` 的情况下, 容器将在运行过程后立即死亡
