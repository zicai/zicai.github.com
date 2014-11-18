---
layout: post
category : 
title: "MinGW 安装使用"
tagline: "Supporting tagline"
tags : [MinGW]
---

##安装
[官网地址](http://www.mingw.org/)

[下载地址](http://sourceforge.net/projects/mingw/files/latest/download?source=files)

安装完之后，在出现的窗口中选择要安装的套件，

- 勾选mingw-developer-toolkit,mingw32-base,mingw32-gcc-g++,msys-base.
- 然后左上角installation，选择apply change
- 将安装路径添加到PATH中
- 测试安装是否成功，打开命令行输入`gcc -v`。注意如果是win8，安装完 MinGW 是没有 `C:\MinGW\bin\gcc.exe` 而是 `C:\MinGW\bin\g++.exe`。cmd指令要改为`g++ -v`

安装完后的配置：

- 切换到C:\MinGW\msys\1.0，双击msys.bat进入MinGW Shell  。输入`mingw-get --help`
- 然后使用`mingw-get install msys-groff` 和`mingw-get install msys-man`命令来安装man包，
- 然后去 [https://www.kernel.org/pub/linux/docs/man-pages/](https://www.kernel.org/pub/linux/docs/man-pages/)网站下载man手册，建议下载最新的版本，放置到c:\MinGW\msys\1.0\home\Administrator目录下，其中"c:\MinGW"是我的 MinGW安装目录，"Administrator"当前登录windows的用户名。切换到该目录`tar -xzvf man-pages-X.XX.tar.gz`解压缩该文件。进入目录man-pages-3.58 执行`make`
man手册生成成功后，执行 `man printf`,测试一下。