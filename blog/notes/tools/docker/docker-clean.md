# 清理占用的磁盘空间

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
