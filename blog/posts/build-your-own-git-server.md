# 内网服务器搭建 GitLab 代码仓库

{docsify-my-updater}

?> 参考资料：  
gitlab-ce (官方 docker 镜像)：https://docs.gitlab.com/ee/install/docker.html  
gitlab-ce-zh (第三方汉化 docker 镜像)：https://github.com/twang2218/gitlab-ce-zh



## 前言

为了使用私有仓库，所以决定在公司内网搭建 Git 私服。而 GitLab 是目前最流行且功能齐全的方案，集成了 CI/CD，提供了开箱即用的 Docker 镜像。



## 安装

推荐使用 Docker 方式部署。

我们先假设，内网主机服务器 IP 是 192.168.1.101，我希望使用 9980 端口作为 Web 界面的访问端口（假设 80 端口已被占用或不能使用），部署后的访问地址就是：http://192.168.1.101:9980 。

那么，下面开始吧 XD



### 使用 Docker 命令启动

```terminal
$|docker run -d \
    --name gitlab \
    --restart always \
    -p 9980:9980 \
    -p 9922:22 \
    -p 9943:443 \
    -v /home/dockerapps/gitlab/config:/etc/gitlab \
    -v /home/dockerapps/gitlab/data:/var/opt/gitlab \
    -v /home/dockerapps/gitlab/logs:/var/log/gitlab \
    --env GITLAB_OMNIBUS_CONFIG="external_url 'http://192.168.1.101:9980';" \
    twang2218/gitlab-ce-zh:11.1.4
```

运行后，GitLab 服务就起来啦，首次访问登录必须设置管理员(root)密码。

数据都我映射在 `/home/dockerapps/gitlab` 路径下，目录结构如下：

```
gitlab
|-- config
|     |-- gitlab.rb  # 配置文件
|-- data
|     |-- backups    # 备份文件
|-- logs
```

**下面是关于命令的一些说明。**

你可能会注意到，为什么是主机映射到容器内部都是 9980 端口，而不是文档上说的 80 端口？

```bash
# 为什么不是指定到容器内的 80
-p 9980:80
```

因为当我要使用非 80 端口时，主机和容器的端口要一致。[原因看这里](https://docs.gitlab.com/ee/install/docker.html#expose-gitlab-on-different-ports)

```bash
# 主机和容器都是 9980
-p 9980:9980
```

同时和 `external_url` 设置的端口也一致才能生效。因为这一步我已经通过 `--env GITLAB_OMNIBUS_CONFIG` 传入配置了，所以不需要在启动后去修改配置文件。

但这里我还是写一下修改配置文件的步骤：

用编辑器打开 `/home/dockerapps/gitlab/config/gitlab.rb` 文件，并设置 `external_url` 

```bash
# For HTTP
external_url "http://192.168.1.101:9980"
```

如果要修改 HTTPS(443) 和 SSH(22) 默认端口也是一样的：

```bash
# For HTTPS(443)
# 先指定映射端口 -p 9943:9943
external_url "http://192.168.1.101:9943"

# For SSH(22)
# 先指定映射端口 -p 9922:9922，再设置 gitlab_shell_ssh_port
gitlab_rails['gitlab_shell_ssh_port'] = 9922
```

最后，重新配置 GitLab：

```bash
docker exec -it gitlab gitlab-ctl reconfigure
```



### 使用 Docker Compose

相比命令方式，有了配置文件会更方便管理。

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
        external_url 'http://my.domain.com'
        gitlab_rails['time_zone'] = 'Asia/Shanghai'
    ports:
      - "9980:9980"
      - "9922:22"
      - "9943:443"
    volumes:
      - /home/dockerapps/gitlab/config:/etc/gitlab
      - /home/dockerapps/gitlab/data:/var/opt/gitlab
      - /home/dockerapps/gitlab/logs:/var/log/gitlab
```

启动服务

```bash
docker-compose up -d
```

停止服务

```bash
docker-compose down
```



## 备份

使用 gitlab-rake 备份数据

```terminal
$|gitlab-rake gitlab:backup:create
```

默认备份至 /var/opt/gitlab/backups 路径下，如果按照我的配置启动，外挂路径是 /home/dockerapps/gitlab/data/backups

另外需要手动备份以下配置文件：

- /etc/gitlab/gitlab-secrets.json

- /etc/gitlab/gitlab.rb

- /home/git/gitlab/config/secrets.yml



## 恢复

在恢复之前需要首先停止服务：

```terminal
$|gitlab-ctl stop
```

备份文件必须在 /var/opt/gitlab/backups 路径下（我的外挂路径是 /home/dockerapps/gitlab/data/backups），假如名称为 1597188417_2020_08_11_12.10.5_gitlab_backup.tar，不需要加上 _gitlab_backup.tar 后缀。

执行下面的恢复命令：

```terminal
$|gitlab-rake gitlab:backup:restore BACKUP=1597188417_2020_08_11_12.10.5
```



## 资源限制

GitLab 的 CPU、内存占用是很高的，如果是小团队或个人使用，用户数不多会造成资源浪费。所以我们可以通过一些配置，限制资源过高的情况发生。

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

经实测，在我的 1 核 2 G云服务器上也有 10% 左右剩余内存（上面跑了宝塔面板和 gitlab）。不过我个人的 git 服务器并不会用到太多功能，因此我后来迁移到了 gitea，内存占用直降到 60%，这就是后话了。