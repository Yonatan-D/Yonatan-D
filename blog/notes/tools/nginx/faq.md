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
