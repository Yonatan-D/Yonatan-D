## 使用 curl 代替 telnet 测试端口是否连通

```sh
curl -vv telnet://172.16.1.1:8080
```

## curl 下载 github release 文件

`-L`参数会让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。

`-O`参数将服务器回应保存成文件，并将 URL 的最后部分当作文件名。

```bash
curl -LO https://github.com/Yonatan-D/OnlineSurvey/releases/download/v1.0.0/onlinesurvey-v1.0.0.tar.gz
```

国内加速下载 (https://gh-proxy.com/) : 

```bash
curl -O https://gh-proxy.com/https://github.com/Yonatan-D/OnlineSurvey/releases/download/v1.0.0/onlinesurvey-v1.0.0.tar.gz
```

## curl 获取文件大小 （不下载）

curl -I 或者 curl --head 能获取到返回的头信息
同样, wget --spider

```bash
curl -LI https://github.com/Yonatan-D/OnlineSurvey/releases/download/v1.0.0/onlinesurvey-v1.0.0.tar.gz | grep -i Content-Length
```