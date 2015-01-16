---
layout: post
title: Sublime 下的 Django
categories: [python]
tags: [sublime, python]
description: Sublime 最近很火啊有木有。 我对 Vim，Emacs 自始至终都没感过冒，但是一用上 Smblime ，我次奥，写起代码来有如神助啊。
---

Sublime 最近很火啊有木有。 我对 Vim，Emacs 自始至终都没感过冒，但是一用上 Smblime ，我次奥，写起代码来有如神助啊。

##1. 安装 Sublime Text 2

这玩意是收费的，虽然不买 License 除了定时弹几个窗口提醒你购买之外没什么影响，但是正当我们爽着的时候，突然这么一下还是很扫兴的。感谢115，需要就点这里吧，你懂的。

##2. 安装 Package

抄自网上的一段，感谢不知道被转载了多少次的无名作者。

1. 打开 Sublime Text 2，按下 Control + ` 调出 Console

2. 将以下代码粘贴进命令行中并回车：

		import urllib2,os;pf=’Package Control.sublime-package’;ipp=sublime.installed_packages_path();os.makedirs(ipp) if not os.path.exists(ipp) else None;open(os.path.join(ipp,pf),’wb’).write(urllib2.urlopen(‘http://sublime.wbond.net/’+pf.replace(‘ ‘,’%20′)).read())

3. 重启 Sublime Text 2，如果在 Preferences -> Package Settings 中见到 Package Control 这一项，就说明安装成功了。

##3. 安装插件 

我只用了少量插件，个人感觉这些都是必装的。

1. Ctags

	函数跳转插件。用过 Vim 的应该比较熟悉，像我这样被 Eclipse 惯坏的可能会觉得函数跳转还得装插件，是件不可思议的事。

	首先在 Sublime 里 Control + Shift + P 打开控制面板，找到 Package Control: Install Package，会列出所有可装的插件。 输入“ctags”，回车，马上就装好了。

	Ctags 插件实际上是执行系统中的”ctags -R -f .tags”命令。关于 OS X 中的 ctags 命令，这里比较纠结，其实系统里已经有了，但是不支持“-R”参数，因此我们必须安装另一版本的 Ctags。

		$ sudo port install ctags
	
	安装好之后，新 ctags 的路径为：

		/opt/local/bin/
	打开我们的 Sublime 编辑器，找到菜单 Preference – Package Setting – Ctags – Settings Default，将其中的“extra_tag_paths” 属性的值改为：

		[ [["source.python", "windows"], “/opt/local/bin”]],
	现在我们随便打开一个 Java 项目测试一下，在项目上点右键（双指触摸），点击 Ctags: Rebuild Tags，建立缓存。

	在函数上按住 Option + Shift  + 鼠标左键（单指触摸）就能实现跳转啦。
	
2. GBK Encoding Support

	中文支持插件。没什么好说的，装！

3. Git

	版本控制插件。先在 Sublime 中安装 Git 插件，之后回到终端：

		$ sudo port install git-core
		
	在天朝速度貌似比较慢，等着吧。装完打开菜单中 Tools – Git 试试是不是好用。

4. Sublime CodelIntel

	命令补全插件。装上它，我们的 Django 代码就能自动补全了（大部分）。首先安装 CodeIntel，接着我们还需要一步设置。打开终端：
	
		$ python
		import django
		django
		
	得到 Django 的安装路径
	
		$ cd ~/.codeintel
		$ open .
		
	我们打开了一个隐藏文件夹，紧接着打开里面的 config 文件，将“pythonExtraPaths”属性的值改成：

		/Library/Python/2.7/site-packages
	

