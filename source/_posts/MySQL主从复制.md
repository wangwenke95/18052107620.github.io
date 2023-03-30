---
title: MySQL主从复制
date: 2023-03-30 08:32:26
tags: MySQL主从复制
categories: MySQL
description: 对数据库进行主从复制减轻主服务器的压力，主数据库增删改，从数据库负责查询
keywords: 主从复制,MySQL主从复制，Mysql
cover: https://img1.baidu.com/it/u=1128666124,3756061506&fm=253&fmt=auto&app=138&f=JPEG?w=798&h=500
---
# MySQL主从复制

## 什么是主从复制？

MySQL 主从复制是一种数据复制技术，它允许将一个 MySQL 数据库的更改复制到其他 MySQL 实例中。在主从复制中，有一个主服务器（master）和一个或多个从服务器（slave）。主服务器上的更改会被自动地复制到从服务器上。

主从复制有很多用途，比如：

- 负载均衡：将读操作分配给从服务器，从而减轻主服务器的负担。
- 备份：可以将从服务器作为备份服务器，以便在主服务器故障时恢复数据。
- 分布式数据处理：将数据复制到多个从服务器上进行分布式计算。 

## 如何设置主从复制？

要设置 MySQL 主从复制，需要完成以下步骤：

### 步骤1：创建主服务器
在<font color="#EA4335" size=4px;>主服务器</font>上创建一个 MySQL 数据库，并确保该数据库可以被远程连接。

docker语法参照:
```bash
docker run  -p 3306:3306 
-v /mydata/mysql-master/log:/var/log/mysql 
-v /mydata/mysql-master/data:/var/lib/mysql 
-v /mydata/mysql-master/conf:/etc/mysql 
-e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```
### 步骤2：启用二进制日志记录
在<font color="EA4335" size=4px;>主服务器</font>上启用二进制日志记录。这将使主服务器将所有更改记录到二进制日志文件中。

```properties
# 在 my.cnf 文件中添加以下配置
log-bin=mysql-bin
```
如果你无法找到 my.cnf 文件，可以尝试在终端或命令提示符中执行以下命令，查看 MySQL 的配置文件位置：
```shell
mysql --help | grep "my.cnf"
```
这个命令会返回 MySQL 配置文件的位置信息。如果你仍然无法找到 my.cnf 文件，可以尝试手动创建一个。在 Linux 上，可以使用以下命令创建 my.cnf 文件：
```shell
sudo vim /etc/mysql/my.cnf
```
主服务器my.cnf配置参照:
```shell
[mysqld]

## 设置server_id，同一局域网中需要唯一，不可以和从的重复

server_id=101 

## 指定不需要同步的数据库名称

binlog-ignore-db=mysql  

## 开启二进制日志功能

log-bin=mall-mysql-bin  

## 设置二进制日志使用内存大小（事务）

binlog_cache_size=1M  

## 设置使用的二进制日志格式（mixed,statement,row）

binlog_format=mixed  

## 二进制日志过期清理时间。默认值为0，表示不自动清理。

expire_logs_days=7  

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。

## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致

slave_skip_errors=1062

```

创建好后重启mysql服务：
```shell
service mysql restart 
```

### 步骤3: 数据库创建用户

进入<font color="#EA4335" size=4px;>主服务器</font>的mysql，执行mysql语句创建一个从服务器的用户:
```mysql
GRANT REPLICATION SLAVE ON *.* to 'slave'@'%' identified by 'Root@123456';

```

这里 `'slave'`是用户名 `'Root@123456'`是主服务器mysql的root密码



### 步骤4：创建从服务器
在<font color="#EA4335" size=4px;>从服务器</font>上创建一个 MySQL 数据库，并确保该数据库可以被远程连接。

### 步骤5：配置从服务器
在<font color="#EA4335" size=4px;>从服务器</font>上配置以下选项：

```properties
# 在 my.cnf 文件中添加以下配置
[mysqld]
server-id=2
relay-log=mysql-relay-bin
```

其中，`server-id` 是从服务器的 ID，应该与主服务器的 ID 不同。`relay-log` 是从服务器上的中继日志文件名。

这里如果没有这个文件我们需要自己在对应的目录下创建该文件，一般目录存在/etc/mysql/my.cnf
```shell
[mysqld]

## 设置server_id，同一局域网中需要唯一,不可以和主的重复
server_id=102

## 指定不需要同步的数据库名称

binlog-ignore-db=mysql  

## 开启二进制日志功能，以备Slave作为其它数据库实例的Master时使用

log-bin=mall-mysql-slave1-bin  

## 设置二进制日志使用内存大小（事务）

binlog_cache_size=1M  

## 设置使用的二进制日志格式（mixed,statement,row）

binlog_format=mixed  

## 二进制日志过期清理时间。默认值为0，表示不自动清理。

expire_logs_days=7  

## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。

## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致

slave_skip_errors=1062  

## relay_log配置中继日志

relay_log=mall-mysql-relay-bin  

## log_slave_updates表示slave将复制事件写进自己的二进制日志

log_slave_updates=1  

## slave设置为只读（具有super权限的用户除外）

read_only=1
```


### 步骤6：连接主服务器和从服务器
在<font color="#EA4335" size=4px;>从服务器</font>上执行以下命令：

```sql
CHANGE MASTER TO
    MASTER_HOST='主服务器IP地址',
    MASTER_USER='主服务器创建的那个用户名',
    MASTER_PASSWORD='主服务器mysql密码',
    MASTER_LOG_FILE='主服务器上的二进制日志文件名',
    MASTER_LOG_POS='主服务器上的二进制日志文件的位置';
    MASTER_CONNECT_RETRY='失联后重连秒数';
```
例如:
```sql
CHANGE MASTER TO
    MASTER_HOST='192.168.0.124',
    MASTER_USER='slave',
    MASTER_PASSWORD='Root@123456',
    MASTER_LOG_FILE='mall-mysql-bin.000006',
    MASTER_LOG_POS=438,
    MASTER_CONNECT_RETRY=60;
```
其中，`MASTER_HOST` 是主服务器的 IP 地址，`MASTER_USER` 和 `MASTER_PASSWORD` 是连接主服务器的用户名和密码，`MASTER_LOG_FILE` 和 `MASTER_LOG_POS` 是主服务器上的二进制日志文件名和位置。其中，`MASTER_CONNECT_RETRY` 参数的值可以根据实际需要进行调整。这里将其设置为 60 秒，表示在连接失败后每隔 60 秒尝试重新连接一次。

**可以通过执行以下命令来获取主服务器上正在使用的二进制日志文件名和位置**

在主服务器的mysql中执行:
```sql
SHOW MASTER STATUS;
```
查看状态:
```
+-----------------------+----------+--------------+------------------+-------------------+
| File                  | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+-----------------------+----------+--------------+------------------+-------------------+
| mall-mysql-bin.000006 |      438 |              | mysql            |                   |
+-----------------------+----------+--------------+------------------+-------------------+
```
记住`File`和`Position`


{% note warning simple %}
注意：这里执行完查询命令主服务器就不要再做任何操作，不然Position会有可能改变
{% endnote %}



### 步骤7：启动从服务器
在<font color="#EA4335" size=4px;>从服务器</font>上执行以下命令：

```sql
START SLAVE;
```



```sql

SHOW SLAVE STATUS\G;
```

在输出中查看 `Slave_IO_Running` 和 `Slave_SQL_Running` 字段的值，确保它们都为 Yes。



## 步骤7：测试主从复制
在主服务器上进行一些更改，然后在从服务器上检查这些更改是否已复制。可以使用以下命令检查从服务器的状态，比如在主服务器创建一个数据库和表,增加一条数据，看下从服务器上查看有没有这条数据。


# 总结
MySQL 主从复制是一种常用的数据复制技术，它可以将一个 MySQL 数据库的更改自动复制到其他 MySQL 实例中。主从复制有很多用途，比如负载均衡、备份和分布式数据处理等。

要设置 MySQL 主从复制，需要在主服务器上创建一个 MySQL 数据库，并启用二进制日志记录；在从服务器上创建一个 MySQL 数据库，并配置从服务器选项；连接主服务器和从服务器，然后启动从服务器。可以使用 SHOW SLAVE STATUS 命令检查从服务器的状态。

在使用 MySQL 主从复制时，需要注意以下几点：

- 主服务器和从服务器的 MySQL 版本应该相同。
- 主服务器和从服务器的数据存储引擎应该相同。
- 不要在从服务器上进行写操作，否则可能会导致数据不一致。
- 需要定期备份从服务器上的数据，以防止数据丢失。

希望这个 MySQL 主从复制笔记对你有所帮助。
