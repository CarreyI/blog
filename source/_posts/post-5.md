---
title: jar命令打包与java执行jar包
date: 2016-07-14 17:07:21
categories: Java
---

## jar命令详解
jar {ctxu}[vfm0M] [jar-文件] [manifest-文件] [-C 目录] 文件名 ... 

　　其中 {ctxu} 是 jar 命令的子命令，每次 jar 命令只能包含 ctxu 中的一个，它们分别表示： 

* -c　创建新的 JAR 文件包 
* -t　列出 JAR 文件包的内容列表 
* -x　展开 JAR 文件包的指定文件或者所有文件 
* -u　更新已存在的 JAR 文件包 (添加文件到 JAR 文件包中) 

　　[vfm0M] 中的选项可以任选，也可以不选，它们是 jar 命令的选项参数

* -v　生成详细报告并打印到标准输出 
* -f　指定 JAR 文件名，通常这个参数是必须的 
* -m　指定需要包含的 MANIFEST 清单文件 
* -0　只存储，不压缩，这样产生的 JAR 文件包会比不用该参数产生的体积大，但速度更快 
* -M　不产生所有项的清单（MANIFEST〕文件，此参数会忽略 -m 参数 

### 创建jar包并显示打包过程
``` bash
jar -cvf filename.jar files
```

### 创建可执行jar包并显示打包过程
``` bash
jar -cvfm filename.jar MANIFEST.MF files
```

### 查看jar包中的文件
``` bash
jar -tf filename.jar
```

### 解压jar包并显示打包过程
``` bash
jar -xvf filename.jar
```

### 向jar包中添加文件
``` bash
jar -uf filename.jar files
```

### 加-C参数，表示先切换到TEST目录下在执行jar -cvf命令
``` bash
jar -cvf filename.jar -C TEST/
```
---------
## java执行jar包

### 执行不带MANIFEST文件的jar包
``` bash
java -classpath filename.jar MainClass
```
例如有一个类叫helloworld里边有main方法代码如下：
```
public class helloworld{
    public static void main(String[]args){
	System.out.println("hello world");
    }
}
```
然后将他编译打成jar包，没有指定添加MANIFEST.MF文件或没有执定main方法所在的类，使用java -jar helloworld.jar会报no main manifest attribute, in helloworld.jar错误，这种情况就可以使用java -classpath helloworld.jar helloworld命令指定main class执行

### 执行jar包中包含jar包的jar包
``` bash
java -classpath inner.jar -jar filename.jar
```
有时候我们需要引用第三方的jar包，我们打包的时候就需要把第三方jar包一起打到jar包中，这时候有两种方法：
* 第一种把第三方的jar包解压后与项目一起打包
* 第二种使用上边的命令去执行，例如有一个第三方的jar包叫inner.jar与项目一起打包成jar文件叫helloworld.jar使用java -jar命令去执行会报找不到class异常，就需要使用上边的命令java -classpath inner.jar -jar helloworld.jar

