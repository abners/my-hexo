title: mysql开启数据库远程访问权限
date: 2016-02-15 10:00:00
tags:
- 数据库
- mysql
categories: 技术日志
comments: false
---

当我们在一台机器上新安装好一个mysql服务器后，发现通过远程连接工具（如heidisql）无法访问，总是出现“ERROR 1130: Host 'lijuan-' is not allowed to connect to this MySQL server”这样的错误，这是因为缺省状态下mysql用户没有远程访问的权限，只允许本地进行访问，下面给出两种方式解决这一问题：
<!--more-->
1、修改表法 

帐号初始时不允许从远程登陆，只能在localhost。这个时候只要在localhost所在的主机，使用root用户登入mysql后，更改 "mysql" 数据库里的 "user" 表里的 "host" 项，将"localhost"改为"%" 

```
[root@iZ25fz36336Z mysql]# mysql -uroot -p 

mysql>use mysql; 

mysql>update user set host = '%' where user = 'root'; 

mysql>select host, user from user; 

mysql>FLUSH PRIVILEGES //改完之后需要刷新下用户的权限，否则不会生效
```
2、通过修改授权进行更改

在安装mysql的机器上运行： 
```
1、[root@iZ25fz36336Z mysql]# mysql -uroot -p  //使用root用户登录mysql服务器 

2、mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION  //赋予任何主机访问数据的权限 

   例如，如果想使用test用户和mypassword这个密码从任何主机连接到mysql服务器的话： 

   GRANT ALL PRIVILEGES ON *.* TO 'test'@'%'IDENTIFIED BY 'mypassword' WITH GRANT OPTION; 

   如果想允许用户test从ip为192.168.1.16的主机连接到mysql服务器，并使用mypassword作为密码： 

   GRANT ALL PRIVILEGES ON *.* TO 'test'@'192.168.1.16' IDENTIFIED BY 'mypassword' WITH GRANT OPTION; 

3、mysql>FLUSH PRIVILEGES  //修改生效 

4、mysql>EXIT 

```
退出MySQL服务器，这样就可以在其它任何的主机上以root身份登录了