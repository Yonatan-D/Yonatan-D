# GitLab

## 安装使用

### 安装

<!-- tabs:start -->

#### **docker compose**

```yaml
# mkdir -p /apps/gitlab && vim /apps/gitlab/docker-compose.yml
version: '2'
services:
  gitlab:
    image: twang2218/gitlab-ce-zh:11.1.4
    container_name: gitlab
    restart: always
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://192.168.0.1:8880'
        gitlab_rails['time_zone'] = 'Asia/Shanghai'
    ports:
      - "8880:8880"
      - "8081:443"
      - "8082:22"
    volumes:
      - /apps/gitlab/config:/etc/gitlab
      - /apps/gitlab/data:/var/opt/gitlab
      - /apps/gitlab/logs:/var/log/gitlab
```

#### **docker**

```bash
docker run -d \
  --name gitlab \
  --restart always \
  -p 8880:8880 \
  -p 8081:443 \
  -p 8082:22 \
  -v /apps/gitlab/config:/etc/gitlab \
  -v /apps/gitlab/data:/var/opt/gitlab \
  -v /apps/gitlab/logs:/var/log/gitlab \
  --env GITLAB_OMNIBUS_CONFIG="external_url 'http://192.168.0.1:8880';" \
  twang2218/gitlab-ce-zh:11.1.4
```

<!-- tabs:end -->

### 修改时区

默认时区是 UTC，创建和提交信息会有 8 小时时差

修改时区后只对新增的提交有效，之前的时间不会变回来

```bash
vim /apps/gitlab/config/gitlab.rb

# gitlab_rails['time_zone'] = 'UTC'
# 修改为
gitlab_rails['time_zone'] = 'Asia/Shanghai'

docker exec -it gitlab gitlab-ctl reconfigure
docker exec -it gitlab gitlab-ctl restart
```

### 备份

```bash
docker exec -it gitlab gitlab-rake gitlab:backup:create
```

默认备份至 `/var/opt/gitlab/backups` 路径下，对应的宿主机路径是：`/apps/gitlab/data/backups`

另外，需要手动备份以下配置文件：

/etc/gitlab/gitlab-secrets.json

/etc/gitlab/gitlab.rb

/home/git/gitlab/config/secrets.yml

### 恢复

```bash
# 恢复备份前，先停止 gitlab 服务
docker exec -it gitlab gitlab-ctl stop

# 把备份文件 1597188417_2020_08_11_12.10.5_gitlab_backup.tar 放到宿主机的 /apps/gitlab/data/backups 路径下
docker exec -it gitlab gitlab-rake gitlab:backup:restore BACKUP=1597188417_2020_08_11_12.10.5
```

### 修改 root 密码

```bash
docker exec -it gitlab gitlab-rails console
u=User.find(1)
u.password='new_password'
u.save!
exit

docker restart gitlab
```

### 限制资源

```bash
# 1. 用编辑器打开配置文件
vim /apps/gitlab/config/gitlab.rb

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

## Pages

### 开启 GitLab Pages 功能

编辑 /etc/gitlab/gitlab.rb 文件，并添加以下行：

```
pages_external_url 'https://your-custom-domain.com/'
gitlab_pages['enable'] = true
```

重启 GitLab 服务：

```
gitlab-ctl restart
```

现在，您应该能够通过 https://your-custom-domain.com/ 访问您的 GitLab Pages 网站。

pages_external_url 设置 Pages 使用的域名，也可以使用主机名，之后可能通过配置 Nginx 实现自定义域名。

## Runner

?> 参考文档：  
官方安装: https://docs.gitlab.com/runner/install/docker.html  
挺清晰的安装步骤: https://www.cnblogs.com/qulianqing/p/9156112.html  

### Docker

安装 runner

```bash
docker run -d --name gitlab-runner --restart always \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:latest
```

注册 runner

```bash
docker exec -it gitlab-runner bash
gitlab-runner register
```

gitlab runnrt 耗性能，如果挂了起不来，重启 docker 也不起效，在 runner 容器里执行：

```bash
gitlab-ci-multi-runner restart
```

### Linux

安装 runner

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash
sudo yum install -y gitlab-runner
```

> https://docs.gitlab.com/runner/install/linux-repository.html

注册 runner

root 账号打开 Runners 管理页面(http://192.168.0.1:8880/admin/runners) ，可以看到 URL 和 Token，该命令代表用 shell 执行方式注册一个 Runners

```bash
gitlab-runner register --non-interactive --executor 'shell' --url 'http://192.168.0.1:8880' --registration-token 'your_registration_token
```

### Windows

下载 GitLab Runner 安装包，https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-windows-amd64.exe

安装成服务，稍后可以在服务找到 “gitlab-runner”

```bash
./gitlab-runner.exe install
```

安装成功会生成 config.toml 文件，打开修改

```toml
# 将 pwsh 修改成 bash （需要先安装 Git 并配置好环境变量）
shell = "bash"
```

启动服务

```bash
./gitlab-runner.exe start
```

注册 runner

root 账号打开 Runners 管理页面(http://192.168.0.1:8880/admin/runners) ，可以看到 URL 和 Token，该命令代表用 shell 执行方式注册一个 Runners

```bash
./gitlab-runner.exe register --non-interactive --executor 'shell' --url 'http://192.168.0.1:8880' --registration-token 'your_registration_token
```

### gitlab-runner 在 dind 方式下使用缓存

```toml
[runners.docker]
  volumes = ["/cache","/run/docker.sock:/run/docker.sock"]
```

参考：https://segmentfault.com/q/1010000022379261

## CI

### 1.提交标签时触发脚本

```yaml
# .gitlab-ci.yml

stages:
  - release

release-to-linux:
  stage: release
  rules:
    # 只有发布标签时才触发
    - if: $CI_COMMIT_TAG != null
  script:
    - echo "Building in Linux..."
    # 执行 linux 环境下的构建脚本
    - bash release_linux.sh $CI_COMMIT_TAG
  tags:
    - linux

release-to-windows:
  stage: release
  rules:
    # 只有发布标签时才触发
    - if: $CI_COMMIT_TAG != null
  script:
    - echo "Building in Windows..."
    # 执行 windows 环境下的构建脚本
    - bash release_win.sh $CI_COMMIT_TAG
  tags:
    - windows
```

### 2.缓存 yarn，避免每次构建

yarn cache: https://classic.yarnpkg.com/en/docs/install-ci/

使用 node 镜像

```yaml
# .gitlab-ci.yml
image: node:9.11.1

before_script:
  - yarn install --cache-folder .yarn

test:
  stage: test
  cache:
    paths:
    - node_modules/
    - .yarn
```

如果使用的是没有安装 yarn 的 docker 镜像，需要安装 yarn

```yaml
# .gitlab-ci.yml
image: does-not-have-yarn

before_script:
  # Install yarn
  - curl -o- -L https://yarnpkg.com/install.sh | bash
  # Add yarn to the path
  - export PATH="$HOME/.yarn/bin:$HOME/.config/yarn/global/node_modules/.bin:$PATH"
```
