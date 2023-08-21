# Gitea 的安装、备份与恢复

{docsify-my-updater}

## 安装

```bash
docker run -dt \
  --name gitea \
  --restart always \
  -p 39248:3000 \
  -p 39222:22 \
  -v /home/gitea:/data \
  -v /etc/timezone:/etc/timezone \
  -v /etc/localtime:/etc/localtime \
  gitea/gitea:latest
```

不指定数据库类型默认使用 SQLite，更节省空间

创建管理员帐户是可选的。第一个注册用户将自动成为管理员



## gitea 首页直接进入登录页面

首先找到自定义配置目录 custom ：

```bash
echo $GITEA_CUSTOM

# docker:
# bash-5.1# echo $GITEA_CUSTOM
# /data/gitea
```

编辑 `/data/gitea/conf/app.ini` 配置文件，添加如下：

```ini
[server]
LANDING_PAGE = explore
```

参考自官网文档：

> ##### 配置说明
>
> 你的所有改变请修改 custom/conf/app.ini 文件而不是源文件。所有默认值可以通过 app.example.ini 查看到
>
> 配置文件：`custom/conf/app.ini` [配置说明](https://docs.gitea.io/zh-cn/config-cheat-sheet/) 
>
> - `LANDING_PAGE`: 未登录用户的默认页面，可选 `home` 或 `explore`。
>
> ##### 自定义 Gitea 配置
>
> Gitea 引用 `custom` 目录中的自定义配置文件来覆盖配置、模板等默认配置。
>
> 如果从二进制部署 Gitea ，则所有默认路径都将相对于该 gitea 二进制文件；如果从发行版安装，则可能会将这些路径修改为Linux文件系统标准。Gitea 将会自动创建包括 `custom/` 在内的必要应用目录，应用本身的配置存放在 `custom/conf/app.ini` 当中。**在发行版中可能会以 `/etc/gitea/` 的形式为 `custom` 设置一个符号链接**
>
> - [快速备忘单](https://docs.gitea.io/en-us/config-cheat-sheet/)
> - [完整配置清单](https://github.com/go-gitea/gitea/blob/master/custom/conf/app.example.ini)
>
> **如果您在 binary 同目录下无法找到 `custom` 文件夹，请检查您的 `GITEA_CUSTOM` 环境变量配置**， 因为它可能被配置到了其他地方（可能被一些启动脚本设置指定了目录）。



## Gitea 备份 (dump)

gitea 在 docker 下备份，遇到一个报错：

```bash
2023/08/04 16:03:46 cmd/dump.go:149:fatal() [F] Unable to open gitea-dump-1691136226.zip: open gitea-dump-1691136226.zip: permission denied
```

没有文件夹权限，尝试 chmod 777 -R /tmp 不管用，通过 gitea dump -h 查询得知以下参数：

```bash
--file value, -f value         Name of the dump file which will be created. Supply '-' for stdout. See type for available types. (default: "gitea-dump-1691139984.zip")
--tempdir value, -t value      Temporary dir path (default: "/tmp")
```

完整操作：

```bash
# 先进入容器内
docker exec -it gitea bash

# 备份到 /backup
mkdir -p /backup/tmp
su git
gitea dump -t /backup/tmp -f /backup/gitea-dump-$(date +"%Y-%m-%d").zip
```



## Gitea 恢复 (restore)

gitea 还没有恢复命令，手动操作如下：

```bash
unzip gitea-dump-2023-08-04
mv /data/gitea/ /data/gitea.bak # 备份原文件
mv data /data/gitea
mv repos/* /data/git/repositories/

sqlite3 $DATABASE_PATH < gitea-db.sql
```

重启 docker 容器，使之生效