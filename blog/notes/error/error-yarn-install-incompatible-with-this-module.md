# yarn install 报错 is incompatible with this module

报错信息：

```sh
fsevents@1.2.13: The platform "win32" is incompatible with this module
# 或
@yarnpkg/core@2.4.0: The engine "node" is incompatible with this module. Expected version ">=10.19.0". Got "10.16.0"
```

报了个 ">=10.19.0". Got "10.16.0"。当前 node 版本 10.16.0 低了，需要升级到 10.19.0 以上版本（yarn 版本对 node 版本有要求）

如果不想升级 node 版本，可以选择忽略这个报错。解决办法：

```sh
yarn config set ignore-engines true
```
