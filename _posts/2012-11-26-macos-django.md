---
layout: post
title: OS X 10.8.2 Django 环境搭建
categories: [Python]
tags: [Mac, Python, Django]
description: 最近工作之余（包括偷懒的时候）对 Python 和 Django 产生了浓厚的兴趣。闲来无事说说环境怎么搭建的吧。
---
最近工作之余（包括偷懒的时候）对 Python 和 Django 产生了浓厚的兴趣。闲来无事说说环境怎么搭建的吧。

##1. 安装 MacPorts

终端下安装软件的工具，还算比较方便，环境搭建的开始。

下载地址：http://www.macports.org/install.php

##2. 安装 Python

OS X 下已经默认安装了 Python 2.7.2 。要是你实在蛋疼了还想安装其他版本，那么在上一步完成的情况下，打开终端：

	$ sudo port
	
正常情况下输入账户密码，应该能进到 port 中了，进不去的多扶老奶奶过马路吧。

看看我们能装的 Python 版本，终端输入：

	search python
	
出来很多版本，选一个安装吧：

	install python25
##3. 安装 Django

到了我们的主角 Django 了，依旧是先下载。

下载地址：https://www.djangoproject.com/download/

之后解压到任意目录，打开终端，执行安装程序：

	$ sudo python setup.py install
	
安装完毕后，我们来验证一下 Django 是否安装成功了。

	$ python

	import django

	django
	
如果没有报错，那么一切顺利，Django 就算安装完毕了。

我们最后执行的一条语句就是 Django 在系统中的绝对路径，记住怎么获得这个路径吧，它在今后还有用处的哦。
