---
title: 在windows以服务的方式启动springboot
date: 2018-11-30 11:20:34
tags:
---

了解一下：
[Windows service wrapper ](https://github.com/kohsuke/winsw)
这个软件可以把任何程序作为服务运行，首先下载exe执行文件，我的执行文件命名:customs.exe
创建customs.xml
```xml
<service>
    <id>Customs</id>
    <name>Customs</name>
    <description>This runs Spring Boot Customs as a Service.</description>
    <executable>java</executable>
    <arguments>-Xmx256m -jar  -Dspring.profiles.active=test "c:\jenkins\soft\customs\customs.jar"</arguments>
    <logmode>rotate</logmode>
</service>
```
然后执行安装服务命令
```
C:\jenkins\soft\customs>customs install
2018-11-30 11:11:23,416 INFO  - Installing the service with id 'Customs'

```
启动服务
```
C:\jenkins\soft\customs>customs start
2018-11-30 11:12:42,031 INFO  - Starting the service with id 'Customs'

C:\jenkins\soft\customs>dir
 驱动器 C 中的卷没有标签。
 卷的序列号是 BCFA-AA62

 C:\jenkins\soft\customs 的目录

2018/11/30  11:12    <DIR>          .
2018/11/30  11:12    <DIR>          ..
2018/11/30  11:12                 0 customs.err.log
2018/11/30  11:06           360,448 customs.exe
2018/11/30  10:37        39,628,558 customs.jar
2018/11/30  11:12               978 customs.out.log
2018/11/30  11:12             1,612 customs.wrapper.log
2018/11/30  11:09               367 customs.xml
```

其中customs.out.log 是spring boot 的执行日志，customs.err.log是错误日志
customs.wrapper.log是service wrapper 的执行日志

看到目录名是jenkins开头的，其实这个项目是用jenkins做CI的，在windows节点执行，要求windows节点接入的时候用管理员身份执行，项目用maven打包完成后添加一个windows shell任务
shell脚本如下：
```
rem copy jar to c:\jenkins\customs\customs.jar
copy target\customs-0.0.1-SNAPSHOT.jar c:\jenkins\soft\customs\customs.jar
rem 进入工作目录
cd c:\jenkins\soft\customs

rem 停止服务
customs stop

rem 启动服务
customs start
```