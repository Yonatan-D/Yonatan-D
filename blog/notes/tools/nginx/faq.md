## Nginx正向代理https时502

https -> https，SNI问题，要用域名！！！

```nginx
proxy_ssl_server_name on
```

## nginx阻止xss和csrf攻击

XSS攻击

```nginx
# 缺少“X-Content-Type-Options”响应头
add_header X-Content-Type-Options: nosniff;
# 缺少“X-XSS-Protection”响应头
add_header X-XSS-Protection "1; mode=block";
# 缺少“Content-Security-Policy”响应头
add_header Content-Security-Policy: default-src 'self'
```

CSRF攻击：配置referer

```nginx
// 合法的referer,所有来自 yonatan.cn 域名和包含google,baidu的站点
valid_referers none blocked *.yonatan.cn server_names ~\.google\. ~/.baidu\. ;

// 不合法的直接返回403
if ($invalid_referer) {
     return 403;
}

location ^~ /api {
    ...
}
```

## SSL 服务器缺少中间证书

检测：https://myssl.com/ssl.html

证书修复：https://myssl.com/chain_download.html

> 参考：
> 
> [微信小程序真机测试 Provisional headers are shown 问题解决办法](https://blog.csdn.net/beiaidefeng/article/details/107159988)
> 
> [Nginx 配 Let's Encrypt HTTPS 证书 报 错误： 服务器缺少中间证书 问题解决](https://leanote.zzzmh.cn/blog/post/admin/Nginx-Let-s-Encrypt%E8%AF%81%E4%B9%A6-%E6%8A%A5%E9%94%99%E8%AF%AF-%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%BC%BA%E5%B0%91%E4%B8%AD%E9%97%B4%E8%AF%81%E4%B9%A6-%E8%A7%A3%E5%86%B3)

## Nginx报错(400) “The plain HTTP request was sent to HTTPS port“

```nginx
proxy_redirect http:// https://;
```

## nginx配置访问密码

```nginx
auth_basic "输入使用密码";
auth_basic_user_file /usr/local/nginx/conf/htpasswd;
```

## 禁用缓存

```nginx
add_header Cache-Control "no-cache, no-store"
```

在线生成工具：https://tool.oschina.net/htpasswd

本地生成密码：

```sh
# centos: yum install httpd-tools
# unbutun: apt-get install httpd 提示选择一个(只测试过选择apache2可用)apt-get install apache2
htpasswd -c [source] username
New password:
Re-type new password:
Adding password for user coderschool

# htpasswd文件长这样：
# username:password 可添加多个
# username:password:comment
# 例如：admin:$apr1$ubbz6W1b$QmPKpJmUEEBZuVxtc5c.G1 <- 使用htpasswd生成的
```

## nginx 搭建下载服务器

```nginx
location /download {
  autoindex on;
  autoindex_exact_size off;
  autoindex_localtime on;
  alias /opt/test-nginx-images/;
}

# 在 server 块内添加下列配置，解决文件名中文乱码
charset utf-8;
```

## nginx防sql注入/溢出攻击/spam及禁User-agents配置

```nginx
server { 
## 禁SQL注入 Block SQL injections 
set $block_sql_injections 0; 
if ($query_string ~ "union.*select.*(") { 
set $block_sql_injections 1; 
} 
if ($query_string ~ "union.*all.*select.*") { 
set $block_sql_injections 1; 
} 
if ($query_string ~ "concat.*(") { 
set $block_sql_injections 1; 
} 
if ($block_sql_injections = 1) { 
return 444; 
} 
  
## 禁掉文件注入 
set $block_file_injections 0; 
if ($query_string ~ "[a-zA-Z0-9_]=http://") { 
set $block_file_injections 1; 
} 
if ($query_string ~ "[a-zA-Z0-9_]=(..//?)+") { 
set $block_file_injections 1; 
} 
if ($query_string ~ "[a-zA-Z0-9_]=/([a-z0-9_.]//?)+") { 
set $block_file_injections 1; 
} 
if ($block_file_injections = 1) { 
return 444; 
} 
  
## 禁掉溢出攻击 
set $block_common_exploits 0; 
if ($query_string ~ "(<|%3C).*script.*(>|%3E)") { 
set $block_common_exploits 1; 
} 
if ($query_string ~ "GLOBALS(=|[|%[0-9A-Z]{0,2})") { 
set $block_common_exploits 1; 
} 
if ($query_string ~ "_REQUEST(=|[|%[0-9A-Z]{0,2})") { 
set $block_common_exploits 1; 
} 
if ($query_string ~ "proc/self/environ") { 
set $block_common_exploits 1; 
} 
if ($query_string ~ "mosConfig_[a-zA-Z_]{1,21}(=|%3D)") { 
set $block_common_exploits 1; 
} 
if ($query_string ~ "base64_(en|de)code(.*)") { 
set $block_common_exploits 1; 
} 
if ($block_common_exploits = 1) { 
return 444; 
} 
  
## 禁spam字段 
set $block_spam 0; 
if ($query_string ~ "b(ultram|unicauca|valium|viagra|vicodin|xanax|ypxaieo)b") { 
set $block_spam 1; 
} 
if ($query_string ~ "b(erections|hoodia|huronriveracres|impotence|levitra|libido)b") { 
set $block_spam 1; 
} 
if ($query_string ~ "b(ambien|bluespill|cialis|cocaine|ejaculation|erectile)b") { 
set $block_spam 1; 
} 
if ($query_string ~ "b(lipitor|phentermin|pro[sz]ac|sandyauer|tramadol|troyhamby)b") { 
set $block_spam 1; 
} 
if ($block_spam = 1) { 
return 444; 
} 
  
## 禁掉user-agents 
set $block_user_agents 0; 
  
# Don’t disable wget if you need it to run cron jobs! 
#if ($http_user_agent ~ "Wget") { 
# set $block_user_agents 1; 
#} 
  
# Disable Akeeba Remote Control 2.5 and earlier 
if ($http_user_agent ~ "Indy Library") { 
set $block_user_agents 1; 
} 
  
# Common bandwidth hoggers and hacking tools. 
if ($http_user_agent ~ "libwww-perl") { 
set $block_user_agents 1; 
} 
if ($http_user_agent ~ "GetRight") { 
set $block_user_agents 1; 
} 
if ($http_user_agent ~ "GetWeb!") { 
set $block_user_agents 1; 
} 
if ($http_user_agent ~ "Go!Zilla") { 
set $block_user_agents 1; 
} 
if ($http_user_agent ~ "Download Demon") { 
set $block_user_agents 1; 
} 
if ($http_user_agent ~ "Go-Ahead-Got-It") { 
set $block_user_agents 1; 
} 
if ($http_user_agent ~ "TurnitinBot") { 
set $block_user_agents 1; 
} 
if ($http_user_agent ~ "GrabNet") { 
set $block_user_agents 1; 
} 
  
if ($block_user_agents = 1) { 
return 444; 
} 
}
```

## 配置 vim nginx.conf 高亮

```sh
#!/bin/bash
wget http://www.vim.org/scripts/download_script.php?src_id=14376 -O nginx.vim

mv nginx.vim /usr/share/vim/vim80/syntax

echo "au BufRead,BufNewFile /etc/nginx/*,/usr/local/nginx/conf/* if &ft == '' | setfiletype nginx | endif" >> /usr/share/vim/vim80/filetype.vim
```
