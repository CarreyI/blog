---
title: Ubuntu下使用U盘安装Ubuntu
date: 2016-07-18 11:10:15
categories: Ubuntu
---

## 制作U盘启动

### 1. 安装U盘制作工具
```
sudo apt-get install unetbootin
```

### 2. 格式化U盘
* 使用 `sudo df -l` 命令查看U盘盘符，这里假设为/dev/sdb1
* 使用 `sudo umount /dev/sdb1` 命令卸载U盘
* 使用 `sudo mkfs.vfat /dev/sdb1` 命令格式化U盘为fat32格式

### 3. 使用unetbootin制作U盘启动，如下图
![该图显示了如何使用unetbootin制作U启动](/img/post_6/1.jpg)

### 4. 设置U盘启动
重启电脑进入BIOS设置U盘启动

## 安装Ubuntu

### 1. U盘启动后选择Install Ubuntu

### 2. 选择语言
![该图显示了Ubuntu选择语言](/img/post_6/1.png)

### 3. 准备安装Ubuntu
![该图显示了准备安装Ubuntu选项](/img/post_6/2.png)

### 4. 选择安装类型
选择其他选项可以自己分区
![该图显示了选择Ubuntu安装类型](/img/post_6/3.png)

### 5. 手动分区

* /swap 交换分区，一般和你内存一样大就好，交换分区，逻辑分区（Swap分区在系统的物理内存不够用的时候，把硬盘空间中的一部分空间释放出来，以供当前运行的程序使用。那些被释放的空间可能来自一些很长时间没有什么操作的程序，这些被释放的空间被临时保存到Swap分区中，等到那些程序要运行时，再从Swap分区中恢复保存的数据到内存中。）

* / 根分区，一般选择15G即可，ext4文件系统，逻辑分区，如果除介绍的前五个外的分区不独立划分其他分区，则都归于此分区，

* /boot 分区，150M就好(也可以再分大一点避免不必要的麻烦) ext4文件系统。主分区 这个分区包含了操作系统的内核和在启动系统过程中所要用到的文件。

* /usr 分区，100个G（看个人电脑硬盘），ext4文件系统，逻辑分区，这个分区是系统和用户软件安装位置，分区大小看个人情况，建议不要太小

* /home 分区，剩余的空间都分到这个分区，ext4文件系统，逻辑分区，用户文件夹

#### 以下分区可分可不分

* /var/log 分区  1G ext4，逻辑分区，系统日志分区

* /tmp  分区 5G ext4，逻辑分区，临时文件分区

* /opt  分区 1G ext4，逻辑分区， 附加程序存放地方

* /bin  分区 ext4，逻辑分区，绝少划分的分区，存放标准系统实用程序。
![该图显示了Ubuntu手动分区](/img/post_6/4.png)

### 6. 设置用户名和密码

## 安装完成后清理

删除libreoffice
```
sudo apt-get remove libreoffice-common
```

删除Amazon
```
sudo apt-get remove unity-webapps-common 
```

删除基本不用的自带软件
```
sudo apt-get remove thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku  landscape-client-ui-install
```
```
sudo apt-get remove onboard deja-dup
```
禁用访客用户，需要重启
```
sudo sh -c 'printf "[SeatDefaults]\nallow-guest=false\n" >/usr/share/lightdm/lightdm.conf.d/50-no-guest.conf'
```
重新启用访客用户，需要重启
```
sudo rm /usr/share/lightdm/lightdm.conf.d/50-no-guest.conf
```

## 一些好用的快捷键

* CTRL + H : 显示隐藏文件
* SUPER + W : 小窗口显示所有程序
* CTRL + SUPER + D : 显示桌面
* CTRL + ALT + T : 打开终端
