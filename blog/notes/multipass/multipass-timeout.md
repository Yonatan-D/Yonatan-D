# 解决打开实例shell超时

记录一次因为系统升级导致 `multipass` 打开实例shell超时的问题，只对有用，仅供参考。

1. 停止 “Multipass” 服务，并结束掉 “multipassd.exe” 任务

2. 重新启动 “Multipass” 服务，启动有问题的实例，确保 `multipass list` 查看到的实例状态是 `Running`

3. 运行 `multipass info <实例名称>`，获得实例的 IP 地址是 `192.168.23.252`

4. 打开 hosts.ics 文件，修改实例网络配置为 `192.168.23.252`

5. 此时已经恢复正常了，赶紧备份数据
