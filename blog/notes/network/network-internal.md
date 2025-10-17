# 内网开发

## windows 双网卡路由配置

目的：实现同时内外网访问

实现：

- 172.16.x.x -> 192.168.x.x

- 其他 -> 互联网

步骤：

```cmd
# 先移除默认策略
route delete 0.0.0.0

# 添加 172.16 段转发到内网网关
route add 172.16.0.0 mask 255.255.0.0 <内网网关> metric 20 -p

# 其它默认转发到外网网关
route add 0.0.0.0 mask 0.0.0.0 <外网网关> metric 22 -p

# 查看路由表
route print
```