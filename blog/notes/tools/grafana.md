## 服务器资源监控方案：windows+grafana+wmi_exporter

wmi_exporter：http://localhost:9182/metrics

Grafana：http://localhost:3000

Prometheus：http://localhost:9090/

步骤：

1. 安装 grafana-7.5.4.windows-amd64.msi


2. 安装 windows_exporter-0.16.0-amd64.msi


3. 进入 prometheus-2.26.0.windows-amd64 目录，双击启动 prometheus.exe


4. 浏览器访问 `http://localhost:3000` 。进入 grafana 页面，初始账号/密码 为 admin/admin，首次登录会要求重设密码。
5. 在设置中导入 Prometheus 数据源
6. 导入仪表盘，ID为 10467。选择 Prometheus 数据源。成功后即可看到系统监控数据
