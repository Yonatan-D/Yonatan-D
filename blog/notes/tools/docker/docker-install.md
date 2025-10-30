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



## 常用命令

Docker 镜像和容器的常用操作命令

### 查看镜像

```bash
docker images
```

### 拉取镜像

```bash
docker pull 镜像地址
# 官方源有时候速度慢，可以使用国内镜像源
```

### 制作镜像

Docker 镜像制作主要有两种方式 **Dockerfile** 和 **快速制作方式**。接下来以实例演示如何通过这两种方式制作镜像。

#### 1. Dockerfile 制作镜像

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



#### 2. 快速制作镜像

将已有容器创建为镜像

查看容器，找到对应的 CONTAINER ID ， 例如： 15adabd78f9b

```bash
docker commit 15adabd78f9b myubuntu
```



### 删除镜像

```bash
docker rmi 镜像名称或ID
```

### 导入导出镜像

使用 save 和 load 命令

```bash
# 导出镜像
docker save myubuntu:latest > myubuntu.tar
# 导入镜像
docker load < myubuntu.tar
```



### 创建容器

```bash
docker run -dt --name=容器名称 --restart=always -p 主机(宿主)端口:容器端口 镜像名称:版本号
```

-v 宿主机目录:容器目录

* 如果host机器上的目录不存在，docker会自动创建该目录
* 如果container中的目录不存在，docker会自动创建该目录
* 如果container中的目录已经有内容，那么docker会使用host上的目录将其覆盖掉

### 进入容器

```bash
docker exec -it 容器名称 /bin/bash
```

### 查看全部容器

```bash
docker ps -a
```

### 查看正在运行容器

```bash
docker ps
```

### 删除容器

```bash
docker rm 容器名称或ID
```

### 导入导出容器

使用 export 和 import 命令

```bash
# 导出镜像
docker export 15adabd78f9b > ubuntu-15adabd78f9b.tar
# 导入镜像
docker import ubuntu-15adabd78f9b.tar
```

export 不会保留镜像的层级信息，所以大小会比 save 小

### 日志排查

```bash
docker logs xxx --tail 500

docker inspect xxx
tail -n 100 /var/lib/docker/containers/xxx/xxx-json.log
```

### 进程排查

```bash
docker stats --no-stream | sort -rn -k 3
docker exec -it xxx ps -aux
```

### 查询cpu和内存占用

```sh
docker stats --no-stream | sort -rn -k 3
```

--format格式化成表格或JSON

```sh
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
docker stats --no-stream --format "{\"name\":\"{{.Name}}\",\"cpu\":\"{{.CPUPerc}}\",\"mem\":\"{{.MemUsage}}\"}"
```