# GitLab 容器启动报错

报错信息：

```sh
/opt/gitlab/embedded/bin/runsvdir-start: line 24: ulimit: pending signals: cannot modify limit: Operation not permitted
/opt/gitlab/embedded/bin/runsvdir-start: line 37: /proc/sys/fs/file-max: Read-only file system
/opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/json-2.1.0/lib/json/common.rb:156:in `parse': 765: unexpected token at '' (JSON::ParserError)
	from /opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/json-2.1.0/lib/json/common.rb:156:in `parse'
	from /opt/gitlab/embedded/service/omnibus-ctl/check_config.rb:32:in `block in load_file'
	from /opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/omnibus-ctl-0.5.0/lib/omnibus-ctl.rb:193:in `block in add_command'
	from /opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/omnibus-ctl-0.5.0/lib/omnibus-ctl.rb:730:in `run'
	from /opt/gitlab/embedded/lib/ruby/gems/2.4.0/gems/omnibus-ctl-0.5.0/bin/omnibus-ctl:31:in `<top (required)>'
	from /opt/gitlab/embedded/bin/omnibus-ctl:23:in `load'
	from /opt/gitlab/embedded/bin/omnibus-ctl:23:in `<main>'
```

无法执行调整 file-max 的系统限制。解决办法：

```sh
# /etc/sysctl.conf 文件末尾追加 fs.file-max = 2097152
echo fs.file-max = 2097152 | sudo tee -a /etc/sysctl.conf

# 重载配置
docker exec -it gitlab gitlab-ctl reconfigure
```