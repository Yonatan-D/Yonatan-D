# 推荐配置 （daemon.json）

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
  // "graph": "/home/docker"
}
```

谨慎修改 bip 和 graph 配置，否则可能会导致 docker 服务无法启动。注意 json 文件格式，最后一行不要加多余的逗号！

daemon.json 文件配置说明：

- `registry-mirrors` : 设置 docker 中国区官方镜像加速

- `log-driver` : 设置 docker 日志驱动为 json-file

- `log-opts` : 设置 docker 日志文件大小为 10m，文件个数为 3 个。限制日志文件大小和个数，避免日志文件过大导致磁盘空间不足，设置后只对新添加的容器有效

- `bip` : 设置 docker 网桥的 IP 地址段，默认是 172.17.0.1/16。避免和内网服务器 IP 地址冲突

- `graph` : 设置 docker 数据文件存放位置，默认是 /var/lib/docker。建议安装 docker 后马上设置

3. 重启 docker 服务

```bash
systemctl daemon-reload
systemctl restart docker
```

4. 验证配置是否成功

```bash
docker info
```
