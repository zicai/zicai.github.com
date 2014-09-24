---
layout: post
category : scrapy
title: scrapy 安装
tagline: "Supporting tagline"
tags : [scrapy]
---

原文地址：[http://doc.scrapy.org/en/latest/intro/install.html](http://doc.scrapy.org/en/latest/intro/install.html)
##准备工作
1. python 2.7
        
        python --version

2. [lxml](http://lxml.de/) 安装指南：[http://lxml.de/installation.html](http://lxml.de/installation.html)

        pip install lxml

	如果报错：

    	make sure the development packages of libxml2 and libxslt are installed 
则先执行：`apt-get install libxml2-dev libxslt1-dev python-dev` 再重新安装

3. [OpenSSL](https://pypi.python.org/pypi/pyOpenSSL)(除了windows，其他系统均已安装)
4. pip或者easy_install
    
    	apt-get install python-pip

##安装Scrapy
NOTE : Don’t use the python-scrapy package provided by Ubuntu, they are typically too old and slow to catch up with latest Scrapy.
安装方法：[http://doc.scrapy.org/en/latest/intro/install.html#intro-install-platform-notes](http://doc.scrapy.org/en/latest/intro/install.html#intro-install-platform-notes)

 1. Import the GPG key used to sign Scrapy packages into APT keyring:

        sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 627220E7

 2. Create /etc/apt/sources.list.d/scrapy.list file using the following command:

        echo 'deb http://archive.scrapy.org/ubuntu scrapy main' | sudo tee /etc/apt/sources.list.d/scrapy.list

 3. Update package lists and install scrapy-VERSION, replace VERSION by a known Scrapy version (i.e.: scrapy-0.22:

        sudo apt-get update && sudo apt-get install scrapy-VERSION