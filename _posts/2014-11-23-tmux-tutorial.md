---
layout: post
category : 
title: "tmux使用"
tagline: "Supporting tagline"
tags : [linux]
---

##安装
linux

    sudo apt-get install tmux

mac

    brew install tmux 
    

##基本概念
- session. session是一个特定的终端组合。输入tmux就可以打开一个新的session。
- window。window 为session中的终端。
- pane 。pane为一个window分隔出来的各个间隔，即window中的终端。

##基本操作

tmux所有自带命令都默认需要先按Ctrl + b，然后再键入对应的命令

###session 操作
- tmux new -s test 新建session，并命名为test
- tmux ls 列出所有session
- Ctrl+b d - (d)eattch 当前session
- tmux attach [-t sessionname]重新进入某session
- Ctrl+b $ - 重命名当前session

###window 操作
- Ctrl+b c - (c)reate 生成一个新的窗口
- Ctrl+b n - (n)ext 移动到下一个窗口
- Ctrl+b p - (p)revious 移动到前一个窗口.
- Ctrl+b w - 列出所有窗口编号,并进行选择切换.
- Ctrl+b & - 确认后关闭当前window tmux 

###pane 操作
- Ctrl+b " - 将当前window水平划分
- Ctrl+b % - 将当前window垂直划分
- Ctrl+b 方向键 - 在各pane间切换
- Ctrl+b，并且不要松开Ctrl，方向键 - 调整窗格大小
- Ctrl+b x - 关闭当前pane
- Ctrl+b { 或} - 左右pane 交换
- Ctrl+b 空格键 - 采用下一个内置布局 
- Ctrl+b q - 显示pane的编号 
- Ctrl+b o - 跳到下一个pane 

TODO： tmux复制粘贴,脚本化----参见参考文章


让状态栏更强大可以试试 [tmux-powerline](https://github.com/erikw/tmux-powerline)

参考文章：

[http://foocoder.com/blog/zhong-duan-huan-jing-zhi-tmux.html/](http://foocoder.com/blog/zhong-duan-huan-jing-zhi-tmux.html/)