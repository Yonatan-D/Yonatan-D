## mysql> flush-hosts

eggjs 报错, mysql 连不上:

```bash
ERROR 2664 nodejs.SequelizeConnectionError: Host '172.16.0.1' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'
```

同一个ip在短时间内产生太多(超过mysql数据库max_connection_errors的最大值)中断的数据库连接而导致的阻塞

解决方法:

```bash
mysql -u root -p

mysql> flush-hosts;
```

排查代码中是否有地方频繁连接数据库，或者数据库连接池配置有问题

## 运维导入数据库后程序启动报错：The user specified as a definer ('root'@'%') does not exist

现场运维遇到一个问题“我的用户不是 root, 但执行 sql 时报错了 root 用户”，排查到原因是视图的安全性为 definer 时只能创建者才能执行, 改成 invoker 只要执行者有权限就可以执行

参考：https://www.cnblogs.com/jichi/p/11972847.html

## 人大金仓(kingbase)数据库把空字符串当成Null

代码里有段sql是 `xxx字段<>''`，意思是查询该字段不为空字符串的记录，但是人大金仓数据库把空字符串当成Null，所以查询结果为空。

复现步骤：

```sql
SELECT '' = '';
```

输出：false

解决只需修改人大金仓数据库配置：

```toml
ora_input_emptystr_isnull = false
```

## MySQL: Prepared statement needs to be re-prepared

> 很简单的查询一行数据的 sql 语句就抛出异常，不清楚是否和数据库中间件最近更新的特性获取表名字段有关，暂时先通过调整 mysql 配置解决

- 查看配置：

```bash
SHOW VARIABLES LIKE '%table_open_cache%';
SHOW VARIABLES LIKE '%table_definition_cache%';
```

- 执行语句，立即生效

```mysql
SET GLOBAL table_open_cache=16384;
SET GLOBAL table_definition_cache=16384;
```

- 再修改下 my.cnf 文件配置，这样下次启动 mysql 的时候就不会出现这个问题了

```bash
[mysqld]
table_open_cache = 16384
table_definition_cache = 16384
```

## mysql日志时间不对

在 mysql 的配置文件 /etc/my.cnf 中 `[mysqld]` 中增加一条 log_timestamps 的配置

配置生效后和系统时间同步，只对后面的日志生效

```
log_timestamps=SYSTEM
```

## 快速备份还原mysql

mysqldump

```mysql
mysqldump --default-character-set=utf8mb4 --host=192.168.14.229 -uroot -p123456 --opt fxdemo | mysql --host=localhost -uroot -p123456 --default-character-set=utf8mb4 -C newfxdemo
```

继上，大数据量导出优化

先通过下面的语句查询出两个值

```mysql
show variables like 'max_allowed_packet';
show variables like 'net_buffer_length';
```

给 sql 语句后面带上两个参数

```mysql
--max_allowed_packet=67108864 --net_buffer_length=16384
```

第三方工具 OBDB2DB：https://www.cnblogs.com/overblue/p/4636367.html

## MySQL理论内存消耗计算

在线计算工具 http://www.mysqlcalculator.com/

```mysql
global级共享内存:

show variables where variable_name in (
'innodb_buffer_pool_size','innodb_log_buffer_size','innodb_additional_mem_pool_size','query_cache_size'
);

session级私有内存:

show variables where variable_name in (
'tmp_table_size','sort_buffer_size','read_buffer_size','read_rnd_buffer_size','join_buffer_size','thread_stack', 'binlog_cache_size'
);

计算公式：
key_buffer_size + query_cache_size + tmp_table_size + innodb_buffer_pool_size + innodb_additional_mem_pool_size + innodb_log_buffer_size
+ max_connections * (
sort_buffer_size + read_buffer_size + read_rnd_buffer_size + join_buffer_size + thread_stack + binlog_cache_size
)
```

> [阅读原文](https://cloud.tencent.com/developer/article/1536662#:~:text=%E4%B8%BA%E4%BB%80%E4%B9%88MySQL%E5%86%85%E5%AD%98%E5%8D%A0%E7%94%A8%E8%BF%99%E4%B9%88%E5%A4%A7%EF%BC%9F%20for%20InnoDB%20MySQL,%E7%9A%84%E5%86%85%E5%AD%98%E6%B6%88%E8%80%97%EF%BC%8C%E4%B8%80%E8%88%AC%E6%9D%A5%E8%AF%B4%E5%8C%85%E5%90%AB%E4%B8%A4%E7%A7%8D%E5%86%85%E5%AD%98%E3%80%82%20%E8%BF%99%E6%98%AF%20Innodb%20%E5%BC%95%E6%93%8E%E6%9C%80%E9%87%8D%E8%A6%81%E7%9A%84%E7%BC%93%E5%AD%98%EF%BC%8C%E4%B9%9F%E6%98%AF%E6%8F%90%E5%8D%87%E6%9F%A5%E8%AF%A2%E6%80%A7%E8%83%BD%E7%9A%84%E9%87%8D%E8%A6%81%E6%89%8B%E6%AE%B5%E3%80%82%20%E4%B8%80%E8%88%AC%E6%98%AFglobal%E5%85%B1%E4%BA%AB%E5%86%85%E5%AD%98%E4%B8%AD%E5%8D%A0%E7%94%A8%E6%9C%80%E5%A4%A7%E7%9A%84%E9%83%A8%E5%88%86%E3%80%82)

## mysql表结构死锁

```mysql
# 查看 status 列有没有 lock 字眼
show processlist;
kill [id]
# 查看正在运行的事务
SELECT * FROM information_schema.INNODB_TRX;
kill [id]
```

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

以ibd结尾的是：名称对应的独立的表空间文件

以frm结尾的是：名称对应的表结构定义文件

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