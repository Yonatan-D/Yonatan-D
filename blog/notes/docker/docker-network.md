## 白名单 (iptables)

拒绝所有访问

```bash
iptables -I DOCKER-USER -i ens33 -j DROP
```

拒绝对某容器的访问

```bash
iptables -I DOCKER-USER -i ens33 -d 172.17.0.2 -j DROP
```

放行IP对某容器的访问

```bash
iptables -I DOCKER-USER -i ens33 -s 192.168.150.220 -j ACCEPT
```

容器设置固定 IP (容器默认 172.17.x.x 每次重启是可能改变的，它是根据容器启动顺序分配的)

```bash
docker network create --subnet=xxx.xxx.xxx.0/24 newNetWork
docker run -itd --name dockerName --network=newNewWork --ip xxx.xxx.xxx.1 imageName
```

示例：

搭建一个 svn 服务，设置 IP 白名单

```bash
docker network create --subnet=172.18.0.0/24 svn-network
docker run --restart always --name svn -d  --network=svn-network --ip 172.18.0.101 -v /root/svn:/var/opt/svn -p 8081:3690 garethflowers/svn-server

# 拒绝所有IP对该容器的访问
iptables -I DOCKER-USER -i ens33 -d 172.18.0.101 -j DROP
# 放行特定IP对该容器的3690端口的访问
iptables -I DOCKER-USER -i ens33 -s 192.168.150.220 -p tcp -d 172.18.0.101 --dport 3690 -j ACCEPT

# 保存配置
# iptables-save > /etc/sysconfig/iptables
# 或者, 安装了 iptables-services 可以用
service iptables save

# 加载配置
# iptables-restore < /etc/sysconfig/iptables
```

