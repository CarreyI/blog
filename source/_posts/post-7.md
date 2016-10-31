---
title: Ubuntu如何隐藏登录界面用户
date: 2016-07-20 17:20:38
categories: Ubuntu
---

今天在Ubuntu上编译安装了一个mariadb，安装的时候创建了一个mysql用户，结果发现它显示在了登录界面上，让我很不爽，查了一天的资料，都快放弃了最后终于查到了。
官方文档上有写如果是系统用户就会隐藏，在`/var/lib/AccountsService/users/`文件夹下新建一个与用户名相同的文件,在文件中加上
```
[User]
SystemAccount=true
```
标记为系统用户重启就可以了。
