---
title: LAMP安装
date: 2016-12-14 10:35:39
categories: Linux
---

### 1.准备linux环境
*这里以CentOs为例*
下载CentOs [http://mirrors.aliyun.com/centos/](http://mirrors.aliyun.com/centos/)
虚拟机或电脑安装

### 2.安装apache2
下载apache2源码包 [http://httpd.apache.org/download.cgi](http://httpd.apache.org/download.cgi)
放到安装好的服务器

安装依赖
``` shell
yum install gcc prec-devel apr-devel apr-util-devel
```
安装apache2
``` shell
tar -zxvf httpd-2.4.23.tar.gz
cd httpd-2.4.23
./configure --prefix=/usr/apache2 --enable-so --enable-modules=all --enable-mods-shared=all
make && make install
```
安装好后测试是否安装成功
``` shell
/usr/apache2/bin/httpd
```
在浏览器输入ip地址
页面出现It Works!安装成功


配置apache 解析php
*编辑/usr/apache2/conf/httpd.conf文件*
* 添加AddType application/x-httpd-php .php
* 在DirectoryIndex index.html后加上 index.php

加上 ServerName localhost:80 不然启动apache会报AH00558错误

### 3.安装mysql
*这里用Mariadb*
mariadb就不多介绍了，其实它就是mysql
[CentOS下编译安装mariadb](/2016/06/30/post-1/)

### 4.安装php7
下载php7 [http://php.net/downloads.php](http://php.net/downloads.php)
放到安装好的服务器

安装依赖
``` shell
yum install libmcrypt-devel libxml2-devel openssl-devel libcurl-devel libjpeg-devel libpng-devel freetype-devel 
```
安装php7
``` shell
tar -zxvf php-7.1.0.tar.gz
cd php-7.1.0
./configure --prefix=/usr/php7 --with-apxs2=/usr/apache2/bin/apxs --enable-mbstring --with-curl --with-gd --enable-fpm --enable-mysqlnd --with-config-file-path=/usr/php7/etc --with-pdo-mysql=mysqlnd --with-mysql-sock=/var/lib/mysql/mysql.sock --with-openssl --with-zlib --with-iconv --with-mysqli
make && make install
```
安装完之后配置php
``` shell
cp php.ini-production /usr/php7/etc/php.ini
cp /usr/php7/etc/php-fpm.conf.default /usr/php7/etc/php-fpm.conf
cp /usr/php7/etc/php-fpm.d/www.conf.default /usr/php7/etc/php-fpm.d/www.conf
cp sapi/fpm/init.d.php-fpm /etc/init.d/php7-fpm 
chmod +x /etc/init.d/php7-fpm
```
### 测试apache+php
在/usr/apache2/htdocs下新建index.php

添加内容
``` php
<?php
phpinfo();
?>
```
在浏览器输入 ip地址/index.php

