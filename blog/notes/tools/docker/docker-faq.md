# FAQ

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

## 限制docker生成core文件

问题排查：服务器磁盘空间不足，排查发现是docker容器生成的core文件占用了大量空间

```bash
[root /]# ulimit -a
core file size          (blocks, -c) 0		 
# 0不启用, unlited不限制  单位kb

方法一: 修改 docker.service 的 ExecStart 这一行，追加 –default-ulimit core=0:0 # 禁用容器生成Core文件
方法一: docker run --ulimit core=0
```

## docker运行卷映射，宿主机没文件，容器内文件消失

背景：老大做了个 cnpmjs 镜像，我拿到服务器上部署时有个目录做卷映射不成功 (config不可以，download可以)

原因 (网上找到相似经历)：https://www.jianshu.com/p/530d00f97cbf
