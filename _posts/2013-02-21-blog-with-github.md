---
layout: post
category : tutorial
title: github 写博客
tagline: "Supporting tagline"
tags : [github, jekyll, tutorial]
---

1.安装Ruby

    ruby --version
    
2.安装Bundler

    gem install bundler
    
如果无法安装，可能是因为 rubygems.org 被墙，改用[淘宝rubygem镜像](http://ruby.taobao.org/)

    gem sources -l
    gem sources --remove https://rubygems.org
    gem sources -a https://ruby.taobao.org
    gem sources -l

3.安装Jekyll

    gem install github-pages
    
报错如下：

    ERROR:  Error installing github-pages: The 'redcarpet' native gem requires installed build tools.

    Please update your PATH to include build tools or download the DevKit
    from 'http://rubyinstaller.org/downloads' and follow the instructions
    at 'http://github.com/oneclick/rubyinstaller/wiki/Development-Kit'
解决办法：

    到'http://rubyinstaller.org/downloads'下载对应ruby版本的 `DEVELOPMENT KIT`,双击解压。进入解压目录，执行命令
    ruby dk.rb init
    该命令会生成一个config.yml文件，内容是你的ruby安装目录。再执行：
    ruby dk.rb install
    完成安装。
    重新安装Jekyll
    gem install github-pages
4.在博客根目录下创建文件 Gemfile ，在文件中添加

    gem 'github-pages'
5.运行Jekyll，To run Jekyll in a way that matches the GitHub Pages build server, run Jekyll with Bundler . 在项目根目录执行
    
    bundle exec jekyll serve
    这时，应该可以访问
    http://localhost:4000
6.保持Jekyll更新

    bundle update github-pages
    或
    gem update github-pages
    
7.Jekyll[命令文档](http://jekyllrb.com/docs/usage/)

8.切换模板

	rake theme:install name="THEME-NAME"
	rake theme:install git="https://github.com/theme_name.git"

可能出现的问题：

 - 编码错误

        error: invalid byte sequence in GBK.
我用的jekyll版本为1.5.1,解决办法：在 _config.yml 文件里加上
    
        encoding: utf-8


 - 当运行jekyll server 加上 --watch 参数时 报错
    
        in `require': cannot load such file -- wdm (LoadError)
解决办法：在Gemfile中添加。参见 [http://stackoverflow.com/questions/20459859/ruby-error-cannot-load-such-file-wdm-loaderror](http://stackoverflow.com/questions/20459859/ruby-error-cannot-load-such-file-wdm-loaderror)
    
        require 'rbconfig'
        gem 'wdm', '>= 0.1.0' if RbConfig::CONFIG['target_os'] =~ /mswin|mingw|cygwin/i
    
参考文章：
    [https://help.github.com/articles/using-jekyll-with-pages](https://help.github.com/articles/using-jekyll-with-pages)


