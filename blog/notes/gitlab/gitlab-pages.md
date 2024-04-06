## 开启 GitLab Pages 功能

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
