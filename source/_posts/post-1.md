---
title: CentOS下编译安装mariadb
date: 2016-06-30 17:37:34
categories: Mariadb
---
1.    下载CMAKE[https://cmake.org/download/](https://cmake.org/download/)
2.    解压文件`tar -zxvf cmake-3.6.0-rc3.tar.gz`
3.    安装GCC`yum install gcc-c++`
4.    切换到cmake目录`cd cmake-3.6.0-rc3`
5.    执行configure文件`./configure`
6.    安装`make && make install`
7.    下载mariadb[https://downloads.mariadb.org/mariadb/](https://downloads.mariadb.org/mariadb/)
8.    解压文件`tar -zxvf mariadb-10.1.14.tar.gz`
9.    切换到mariadb目录`cd mariadb-10.1.14`
10.   创建mysql用户组及用户
```bash
    groupadd mysql
    useradd -g mysql mysql
```
11.    创建安装目录和数据目录
```bash
    mkdir /usr/local/mariadb 
    mkdir /usr/local/mariadb/data
    chown -R mysql:mysql /usr/local/mariadb
```
12.    安装ncurses-devel`yum install ncurses-devel`
13.    编译`cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mariadb -DMYSQL_DATADIR=/usr/local/mariadb/data -DMYSQL_USER=mysql -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci`
14.    安装`make && make install`
15.    删除旧配置文件复制新文件
```bash
rm -f /etc/my.cnf
cp /usr/local/mariadb/support-files/my-large.cnf /etc/my.cnf
```
16.    初始化数据库
```bash
/usr/local/mariadb/scripts/mysql_install_db --user=mysql
```
17.    添加环境变量
+    运行命令打开系统文件`vi /etc/profile`
+    在最后加上
```bash
MYSQL_HOME=/usr/local/mariadb
PATH=$MYSQL_HOME/bin:$PATH
export $PATH
```
18.    mariadb开机自启动
```bash
cp /usr/local/mariadb/support-files/mysql.server /etc/init.d/mysqld
chkconfig --add mysqld
```
