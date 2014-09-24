---
layout: post
title: "mac 开发环境搭建"
description: ""
category: tutorial
tags: [mac]
---

1. 安装MySQL


    [http://dev.mysql.com/downloads/mysql/](http://dev.mysql.com/downloads/mysql/) 下载mysql

 	安装完成之后，先到系统偏好设置里启动mysql服务

2. 安装mysqldb（mysql-python）

        sudo pip install MySQL-python
        
    如果报错：`mysql_config not found`
    则先执行
    
        export PATH=$PATH:/usr/local/mysql/bin
        
    再安装。
    
    安装完后，在python中使用若报错`Library not loaded:libmysqlclient.18.dylib`,则要做如下修改
    
        sudo ln -s /usr/local/mysql/lib/libmysqlclient.18.dylib /usr/lib
    
3. [http://www.navicat.com.cn/download/navicat-for-mysql](http://www.navicat.com.cn/download/navicat-for-mysql) 下载navicat 
