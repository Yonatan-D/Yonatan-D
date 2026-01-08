# nginx try_files 可以实现先找服务 A，找不到再继续找服务 B

{docsify-my-updater date:2023-05-31}

## 背景

之前遇到过一个问题，项目原来只有一台图片服务器，先把它称为 A 服务器，现在磁盘已经满了，客户那边新加了一台 B 服务器，后面产生的图片都传到 B 服务器，现在希望系统在读取图片时先去找 A 服务器，找不到再去找 B 服务器。

## 原先的 nginx 配置

```nginx
server {
  listen 80;

  location /image {
    proxy_pass http://192.168.1.101:8001;
  }
}
```

## 改进后的配置

其实就是利用了 try_files 和 error_page，当状态码 404 时继续指向下一个服务地址

```nginx
server {
  listen 80;

  location /image {
    try_files $uri @img1;
  }

  location @img1 {
    proxy_pass http://192.168.1.101:8001;
    proxy_intercept_errors on;
    recursive_error_pages on;
    error_page 404 = @img2;
  }

  location @img2 {
    proxy_pass http://192.168.1.102:8001;
  }
}
```