---
layout: post
category : 
title: "Mac 安装 mod_wsgi"
tagline: "Supporting tagline"
tags : [mac]
---

下载源码
[https://github.com/GrahamDumpleton/mod_wsgi/releases](https://github.com/GrahamDumpleton/mod_wsgi/releases)

解压

	tar xvfz mode_wsgi-x.y.tar.gz

切换到解压目录

	./configure
	make
	sudo make install

安装到 /usr/libexec/apache2/mode_wsgi.so

编辑httpd.conf

	vim /etc/apache2/httpd.conf

添加下面

	LoadModule wsgi_module libexec/apache2/mod_wsgi.so

重启apache

	sudo apachectl restart

编译完后清理

	make clean

参考文章：

[https://code.google.com/p/modwsgi/wiki/QuickInstallationGuide](https://code.google.com/p/modwsgi/wiki/QuickInstallationGuide)