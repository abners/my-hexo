title: mysql修改数据存储目录
date: 2016-02-20 15:22:00
tags:
- 数据库
- mysql
categories: 技术日志
comments: true
---


当我们在一台linux服务器上安装好mysql后，其数据库的存储目录通常在/var/lib/mysql/下，其所在目录的磁盘空间通常比较小，为了避免以后因为数据量的增长引起磁盘爆满，从而导致数据库无法正常提供服务，通常我们都会修改数据库的数据存储目录，以下就是修改数据库的数据存储目录的过程。
<!--more-->

1、通常mysql的数据存储目录在/var/lib/mysql/下，但是也可能有之前修改过的情况出现，登入mysql后可以使用show variables命令可以查看你的数据库当前的数据存储目录：

``` 
[root@localhost ~]# mysql -uroot -p //使用root用户登录mysql服务器
mysql> show variables like '%datadir%';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| datadir       | /var/lib/mysql/ |
+---------------+-----------------+

```
2、停止mysql服务器运行

```
[root@localhost /]# service mysqld stop
Stopping mysqld:                                           [  OK  ]
```
3、将上面查到的mysql数据目录移动到你的目标磁盘目录下(以防出现问题，请先将该目录下的文件做下备份)
```
[root@localhost data]# mv /var/lib/mysql /data/myql/
[root@localhost myql]# pwd
/data/myql
[root@localhost myql]# ll
total 4
drwxr-x--x. 5 mysql mysql 4096 Feb 20 08:04 mysql
```
4、修改my.cnf的配置,将datadir的目录改为当前移动到的目录，socket也一样。
```
[root@localhost ~]# vi /etc/my.cnf

# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql  //修改为 datadir=/data/mysql/mysql
socket=/var/lib/mysql/mysql.sock //修改为socket=/data/mysql/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```
5、此时如果直接启动mysql服务，在一些机器上(本人在公司的电脑上)会出现下面这样的错误提示,但是在我自己的电脑上可以启动服务，但是在登录mysql时会出现：
```
[root@localhost mysql]# service mysqld start
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
```
这时只要在/etc/my.cnf中增加如下内容就可以了
```
[mysql] 
　　socket=/data/mysql/mysql/mysql.sock //具体目录要使用你自己的mysql数据目录移动后的路径
```
正文完