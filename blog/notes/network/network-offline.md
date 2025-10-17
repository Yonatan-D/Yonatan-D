# 离线开发

## 下载离线安装包及依赖

-d 仅下载，不安装

```bash
sudo apt clean
sudo apt install -d -y com.alipay.devtools.deepin

mkdir alipayDevtools
cp /var/cache/apt/archives/*.deb alipayDevtools
```

拷贝到离线机上，执行安装命令：

```bash
sudo dpkg -i alipayDevtools/*.deb
```