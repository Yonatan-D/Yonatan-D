## 安装

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

## 备份

```bash
docker exec -it gitlab gitlab-rake gitlab:backup:create
```

默认备份至 `/var/opt/gitlab/backups` 路径下，对应的宿主机路径是：`/apps/gitlab/data/backups`

另外，需要手动备份以下配置文件：

/etc/gitlab/gitlab-secrets.json

/etc/gitlab/gitlab.rb

/home/git/gitlab/config/secrets.yml

## 恢复

```bash
# 恢复备份前，先停止 gitlab 服务
docker exec -it gitlab gitlab-ctl stop

# 把备份文件 1597188417_2020_08_11_12.10.5_gitlab_backup.tar 放到宿主机的 /apps/gitlab/data/backups 路径下
docker exec -it gitlab gitlab-rake gitlab:backup:restore BACKUP=1597188417_2020_08_11_12.10.5
```

## 修改 root 密码

```bash
docker exec -it gitlab gitlab-rails console
u=User.find(1)
u.password='new_password'
u.save!
exit

docker restart gitlab
```

## 限制资源

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