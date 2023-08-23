# 内网服务器搭建 GitLab 代码仓库

{docsify-my-updater}

?> 参考资料：  
gitlab-ce (官方 docker 镜像)：https://docs.gitlab.com/ee/install/docker.html  
gitlab-ce-zh (第三方汉化 docker 镜像)：https://github.com/twang2218/gitlab-ce-zh



## 前言

公司内部不想把产品代码托管在公网上的 Github、Gitee，要求使用私有仓库，于是便在局域网内搭建 Git 私服。我们团队选择使用 GitLab，它是目前最流行且功能齐全的方案，集成了 CI/CD，提供了开箱即用的 Docker 镜像。



## 安装

推荐使用 Docker 方式部署，这里以第三方汉化镜像为例，如果要使用更新版本的官方镜像，链接我贴在了上面。

宿主机信息：

| 用途       | 值                               |
| ---------- | -------------------------------- |
| IP 地址    | 192.168.0.1                      |
| http 端口  | 8080 （80 端口被占用或不能使用） |
| https 端口 | 8081                             |
| ssh 端口   | 8082                             |

部署后访问地址是：http://192.168.0.1:8080 （拉取代码也推荐用 http/https 方式）

那么，下面开始吧~



### 使用 Docker 命令启动

```terminal
$|docker run -d \
    --name gitlab \
    --restart always \
    -p 8080:8080 \
    -p 8081:443 \
    -p 8082:22 \
    -v /home/dockerapps/gitlab/config:/etc/gitlab \
    -v /home/dockerapps/gitlab/data:/var/opt/gitlab \
    -v /home/dockerapps/gitlab/logs:/var/log/gitlab \
    --env GITLAB_OMNIBUS_CONFIG="external_url 'http://192.168.0.1:8080';" \
    twang2218/gitlab-ce-zh:11.1.4
```

运行后，浏览器访问 http://192.168.0.1:8080，首次登录必须设置管理员(root)密码。

按照上述命令，重要的配置、数据、日志文件都映射在宿主机目录： `/home/dockerapps/gitlab` （按你需求更改）

目录文件结构，大致如下：

```
gitlab
|-- config
|     |-- gitlab.rb  # 配置文件
|-- data
|     |-- backups    # 备份文件
|-- logs
```

**下面是关于命令的一些说明。**

> 你可能会注意到，为什么是宿主机映射到容器内部的是 8080 端口，而不是文档上说的 80 端口？
>
> ```bash
> # 【错误示例】为什么不是映射到 80 端口？
> -p 8080:80
> ```
>
> 因为使用非 80 端口时，主机和容器的端口要一致，否则会出错！[点此看官方文档说明](https://docs.gitlab.com/ee/install/docker.html#expose-gitlab-on-different-ports)
>
> ```bash
> # 【正确示例】必须都是 8080 端口！
> -p 8080:8080
> ```
>
> 同时，也要和 `external_url` 设置的端口一致才能生效。因为这一步我已经通过 `--env GITLAB_OMNIBUS_CONFIG` 传入配置了，所以不需要在启动后手动去修改配置文件。
>
> 
>
> 这里留个记录，在宿主机修改配置文件的步骤：
>
> 编辑文件 `/home/dockerapps/gitlab/config/gitlab.rb` ，设置 `external_url` 的值：
>
> ```bash
> # For HTTP
> external_url "http://192.168.0.1:8080"
> ```
>
> 如果要修改 HTTPS(443) 和 SSH(22) 默认端口也是一样的：
>
> ```bash
> # For HTTPS(443)
> # 先指定映射端口 -p 8081:8081
> external_url "https://192.168.0.1:8081"
> 
> # For SSH(22)
> # 先指定映射端口 -p 8082:8082，再设置 gitlab_shell_ssh_port
> gitlab_rails['gitlab_shell_ssh_port'] = 8082
> ```
>
> 最后，让容器内的 gitlab 重载配置：
>
> ```bash
> docker exec -it gitlab gitlab-ctl reconfigure
> ```
>



### 使用 Docker Compose

相比命令方式，使用配置文件更方便管理。

新建 docker-compose.yml，内容如下：

```yaml
version: '2'
services:
  gitlab:
	image: twang2218/gitlab-ce-zh:11.1.4
    container_name: gitlab
    restart: always
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://192.168.0.1:8080'
        gitlab_rails['time_zone'] = 'Asia/Shanghai'
    ports:
      - "8080:8080"
      - "8081:443"
      - "8082:22"
    volumes:
      - /home/dockerapps/gitlab/config:/etc/gitlab
      - /home/dockerapps/gitlab/data:/var/opt/gitlab
      - /home/dockerapps/gitlab/logs:/var/log/gitlab
```

启动服务：

```bash
docker-compose up -d
```

停止服务：

```bash
docker-compose down
```



## 备份

使用 gitlab-rake 备份数据：

```terminal
$|gitlab-rake gitlab:backup:create
```

默认备份至 /var/opt/gitlab/backups 路径下，对应的宿主机路径是：/home/dockerapps/gitlab/data/backups

另外，需要手动备份以下配置文件：

- /etc/gitlab/gitlab-secrets.json

- /etc/gitlab/gitlab.rb

- /home/git/gitlab/config/secrets.yml



## 恢复

恢复备份前，先停止 gitlab 服务：

```terminal
$|gitlab-ctl stop
```

备份文件必须放在 /var/opt/gitlab/backups 路径下，对应的宿主机路径是：/home/dockerapps/gitlab/data/backups

举例，备份文件名称为 1597188417_2020_08_11_12.10.5_gitlab_backup.tar，则不需要加上后缀： _gitlab_backup.tar。执行的恢复命令，如下所示：

```terminal
$|gitlab-rake gitlab:backup:restore BACKUP=1597188417_2020_08_11_12.10.5
```



## 资源限制

GitLab 的 CPU、内存占用很高，如果用户数不多，仅是小团队或个人使用会造成资源浪费。所以我们可以通过一些配置，限制资源过高的情况发生。

```bash
# 1. 用编辑器打开配置文件
vim /home/dockerapps/gitlab/config/gitlab.rb

# 2. 设置
unicorn['worker_processes'] = 2
unicorn['worker_memory_limit_min'] = "100 * 1 << 20"
unicorn['worker_memory_limit_max'] = "250 * 1 << 20"
sidekiq['concurrency'] = 8
postgresql['shared_buffers'] = "128MB"
postgresql['max_worker_processes'] = 4
prometheus_monitoring['enable'] = false

# 3. 重载配置
docker exec -it gitlab gitlab-ctl reconfigure
```

> `prometheus_monitoring['enable'] = false` 是把 Prometheus 给关了，但关闭后也可以选择使用外部 Prometheus。gitlab 服务器已经包含了node-exporter 服务，接口是 http://ip:9100/metrics ，想要在外部使用还必须先改一项配置，打开 `/home/dockerapps/gitlab/data/embedded/cookbooks/monitoring/attributes/default.rb` ，把 localhost:9100 改成 0.0.0.0:9100，然后重载配置 gitlab-ctl reconfigure

实测，在我的 1 核 2 G云服务器上跑了宝塔面板和 gitlab 服务，还有 10% 左右可用内存。不过我个人用不到太多功能，因此后来我替换成 gitea，可用内存多出了 30%。