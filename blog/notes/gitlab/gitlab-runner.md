?> 参考文档：  
官方安装: https://docs.gitlab.com/runner/install/docker.html  
挺清晰的安装步骤: https://www.cnblogs.com/qulianqing/p/9156112.html  

## Docker

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

## Linux

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

## Windows

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
