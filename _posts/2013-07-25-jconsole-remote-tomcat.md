---
layout: post
title: JConsole 远程查看 Tomcat 下的虚拟机
categories: [Java]
tags: [java, tomcat, jconsole]
description: JConsole 远程查看 Tomcat 下的虚拟机
---

##远程

打开的 %tomcat_home%/bin/catalina.sh

配置 JAVA_OPTS，添加代码

	-Dcom.sun.management.jmxremote
	-Dcom.sun.management.jmxremote.port=8091
	-Dcom.sun.management.jmxremote.authenticate=false
	-Dcom.sun.management.jmxremote.ssl=false

命令行下输入 hotstname -i，查看本机的地址是远程访问的地址，比如：192.168.74.138。

##本地

打开 jconsole，新建连接，地址输入：192.168.74.138:8091，就 OK 了。

![图片](/assets/media/2013-07-25-jconsole-remote-tomcat-1.png)