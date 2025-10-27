## node 连接 oracle

环境需要

* oracledb 5.x
* Oracle Instant Client 19.9

首先，项目安装 oracledb：

```js
npm install oracledb
```

然后，本地配置 Oracle Instant Client 环境：

1.下载适用于您平台的 Oracle Instant Client 软件包。下载 Basic 或 Basic Light 软件包

（下载地址：http://www.oracle.com/technetwork/topics/winx64soft-089540.html）

2.将软件包解压缩到一个目录中，如 `D:\oracle\instantclient_19_9`

3.将此目录添加到 `PATH` 环境变量。如果安装了多个版本的 Oracle 库，请确保路径中首先显示的是新目录

4.从 Microsoft 下载并安装正确的 Visual Studio Redistributable。Instant Client 19.9 需要 Visual Studio 2017 redistributable 支持

（下载地址：https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads）

**常见错误**

运行报错，从错误信息中可以看出我们需要安装 Oracle Instant Client：

```
2021-01-12 13:48:10,904 ERROR 28052 nodejs.unhandledRejectionError: DPI-1047: Cannot locate a 64-bit Oracle Client library: "The specified module could not be found". See https://oracle.github.io/node-oracledb/INSTALL.html for help
Node-oracledb installation instructions: https://oracle.github.io/node-oracledb/INSTALL.html
You must have 64-bit Oracle client libraries in your PATH environment variable.
If you do not have Oracle Database on this computer, then install the Instant Client Basic or Basic Light package from
http://www.oracle.com/technetwork/topics/winx64soft-089540.html
A Microsoft Visual Studio Redistributable suitable for your Oracle client library version must be available.
```

如果已经下载并解压了 Oracle Instant Client 软件包还是出现以上报红，请检查：

* 环境变量 `PATH` 的目录路径是否配置正确
* 是否安装了对的 Visual Studio Redistributable 版本，或尝试安装其它版本
* 重启下 Visual Studio Code 软件，使环境变量生效【通常是这个问题】

## mysql大字段 (Row size too large>8126)

```
innodb_file_per_table = 1
innodb_file_format = Barracuda
```

## 数据库名称带 . 符号的

```
如：node.user

解决：`node.user` (用这个引号)
```

## mariadb和mysql对表名大小写敏感

```sql
-- 登入进mysql (0是开启大小写敏感)
show variables like '%table_names';

-- 解决
-- vim /etc/mysql/my.cnf
-- (这里注意[mysqld]下面要空一行，否则可能启动失败！)

[mysqld]

lower_case_table_names=1

-- 记得重启下
service mysqld restart
```

## 导入数据库

```sql
create database xxx;
use xxx;
set names utf8;
source /path/to/xxx.sql
```

## mss2sql

mss2sql下载地址: http://files.cnblogs.com/andrew-blog/mss2sql.rar

参考文章：

https://www.cnblogs.com/angestudy/archive/2012/06/04/2533548.html

[SqlServer数据库如何转换成Mysql的一款工具推荐（mss2sql）_er_916340246的博客-CSDN博客](https://blog.csdn.net/er_916340246/article/details/88892823)

[数据迁移sql server迁移至mysql_zdx355的博客-CSDN博客](https://blog.csdn.net/zdx355/article/details/90295082)

[利用Navicate把SQLServer转MYSQL的方法(连数据) - 麦田守望者~ - 博客园](https://www.cnblogs.com/hgj123/p/10123548.html)

## 查询所有数据库占用磁盘空间大小

```sql
-- sql查询
select 
TABLE_SCHEMA, 
concat(truncate(sum(data_length)/1024/1024,2),' MB') as data_size,
concat(truncate(sum(index_length)/1024/1024,2),'MB') as index_size
from information_schema.tables
group by TABLE_SCHEMA
ORDER BY data_size desc;
-- order by data_length desc;

-- 查看物理文件的路径
show variables like 'datadir'
```

## 启用安全审计

插件：MariaDB Audit Plugin，mysql_plugin_audit_mariadb5.5.66 适用于mysql5.7，参考：[MySQL数据库启用安全审计功能](https://blog.csdn.net/weixin_39699061/article/details/103482490)

```sql
# 查询插件库的路径, 把下载好的 server_audit.dll 放到该目录下
mysql> SHOW GLOBAL VARIABLES LIKE 'plugin_dir';
# 安装审计插件
mysql> INSTALL PLUGIN server_audit SONAME 'server_audit.dll';
# 查看安装结果
mysql> show plugins;
# 查看默认参数配置 (参数server_audit_logging=off，表示审计功能未开启)
mysql> show variables like '%audit%';
```

```sql
# 查询数据库已经安装的插件
mysql> SELECT * from mysql.plugin ;
```

```sql
# 不重启临时开启审计功能
mysql> set global server_audit_logging=on; 
# 永久修改, 编辑 my.ini 文件
```

参数配置

```
server_audit_output_type：指定日志输出类型，可为SYSLOG或FILE
server_audit_logging：启动或关闭审计
server_audit_events：指定记录事件的类型，可以用逗号分隔的多个值(connect,query,table)
参数与MariaDB Audit Plugin插件版本有以下关系：
CONNECT, QUERY, TABLE (MariaDB Audit Plugin < 1.2.0)
CONNECT, QUERY, TABLE, QUERY_DDL, QUERY_DML (MariaDB Audit Plugin >= 1.2.0)
CONNECT, QUERY, TABLE, QUERY_DDL, QUERY_DML, QUERY_DCL (MariaDB Audit Plugin >=1.3.0)
CONNECT, QUERY, TABLE, QUERY_DDL, QUERY_DML, QUERY_DCL, QUERY_DML_NO_SELECT (MariaDB Audit Plugin >= 1.4.4)
server_audit_file_path：如server_audit_output_type为FILE，使用该变量设置存储日志的文件，可以指定目录，默认存放在数据目录的server_audit.log文件中
server_audit_file_rotate_size：限制日志文件的大小
server_audit_file_rotations：指定日志文件的数量，如果为0日志将从不轮转
server_audit_file_rotate_now：强制日志文件轮转
server_audit_incl_users：指定哪些用户的活动将记录，connect将不受此变量影响，该变量比server_audit_excl_users优先级高
server_audit_syslog_facility：默认为LOG_USER，指定facility
server_audit_syslog_ident：设置ident，作为每个syslog记录的一部分
server_audit_syslog_info：指定的info字符串将添加到syslog记录
server_audit_syslog_priority：定义记录日志的syslogd priority
server_audit_excl_users：该列表的用户行为将不记录，connect将不受该设置影响
server_audit_mode：标识版本，用于开发测试
```