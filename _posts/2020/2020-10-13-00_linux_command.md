---
title: linux command
tags: linux
key: 114
article_header:
  type: cover
  image:
    src: https://user-images.githubusercontent.com/8369671/95962914-94f5e600-0e39-11eb-8c74-49bda8ba611d.jpg
---

# Overview
整理一下Linux的命令行类别, 

# [install](https://stackoverflow.com/a/40112859) linux on mac
to make a clean environment for the commands testing 
```
docker run -it --rm ubuntu bash

apt update
apt install vim
```

# cmd<sup>1</sup>
命令行参数一般有两种形式, 即`-`和`--`, 分别表示缩写和全写, e.g.,
```
-r == --recursive
-f == --force
```

## shell vs. bash
- shell, 就是一个程序, 它接受从键盘输入的命令, 然后把命令传递给操作系统去执行
- bash(Bourne again Shell), is one kind of shell, the most widely used one

## 文件系统中跳转
- pwd, 打印出当前工作目录名
- cd, 更改目录
- ls, 列出目录内容

## 操作文件和目录
- cp, 复制文件和目录
- mv, 移动/重命名文件和目录
- mkdir, 创建目录
- rm, 删除文件和目录
- ln, 创建硬链接和符号链接

## 帮助命令
- help/man, 显示命令手册页
- which/whereis, 显示命令在文件系统中的位置
- type, 说明怎样解释一个命令名
- info, 显示命令 info
- whatis, 显示一个命令的简洁描述

## 重定向
- cat, 连接文件
- sort, 排序文本行
- uniq, 报道或省略重复行
- grep, 打印匹配行
- wc, Print newline, word, and byte counts for each FILE
- head, 输出文件第一部分
- tail, 输出文件最后一部分
- tee, 从标准输入读取数据, 并同时写到标准输出和文件(e.g., `ls /usr/bin | tee ls.txt`)

## 键盘操作
- clear, 清屏
- history, cmd历史记录
- 移动光标
   ,  ctl+a, 瞬移到行首
   ,  ctl+e, 瞬移到行尾
- tab, 自动补全

## 权限
0(123)(456)(789)
- 0, 文件类型(-普通文件, d目录, l符号链接等)
- 123, owner, 文件所有者的读\写\可执行权限
- 456, group, 文件组所有者的权限
- 789, others, 其他人的权限

- id, 显示用户身份号
- chmod, 更改文件模式(e.g., `chmod +x xx.sh`)
- su, 以另一个用户的身份来运行shell 
- sudo, 以另一个用户的身份来执行命令 
- adduser, 增加用户
- umask, 设置默认的文件权限
- chown, 更改文件所有者
- chgrp, 更改文件组所有权

## 进程
Linux内核通过使用进程来管理多任务, 
- ps, 报告当前进程快照
- top, 显示任务
- jobs, 列出活跃的任务
- bg, 把一个任务放到后台执行 
- fg, 把一个任务放到前台执行 
- kill, 给一个进程发送信号
- killall, 杀死指定名字的进程 
- shutdown, 关机或重启系统

## 环境
- printenv, 打印部分或所有的环境变量
- set, 设置shell选项(e.g., `set, v`)
- export, 设置session level的环境变量(e.g., `export HISTSIZE=1000`)
- alias, 创建命令别名
- source, 使配置文件生效
   ,  优先级, /etc/profile > ~/.bash_profile(如果有) > ~/.bash_login(如果有) > ~/.profile

## VIM
- ctl+f, 下一页 
- ctl+b, 上一页
- G, 文件尾
- shift+G, 文件头
- number+G, 第number行

## 网络
- ping, 发送ICMP ECHO_REQUEST数据包到网络主机(e.g., `ping google.com`)
- telnet, 链接到网络主机的具体端口(e.g., `telnet google.com 80`)
- traceroute, 打印到一台网络主机的所需的路由跳数(e.g., `traceroute google.com`)
- netstat, 打印网络连接，路由表，接口统计数据，伪装连接，和多路广播成员
- ftp, 因特网文件传输程序
- wget, 非交互式网络下载器
- ssh, OpenSSH SSH客户端(远程登录程序)
- scp, 远程复制

## 查找
- find, 在一个目录层次结构中搜索文件(e.g., `find ./Downloads -type f -name "*.png" | xargs ls -l`)
- xargs, 从标准输入生成和执行命令行
- touch, 更改文件时间
- stat, 显示文件或文件系统状态

## 归档和备份
压缩
- gzip, 压缩或者展开文件 
- bzip2, 块排序文件压缩器

归档, 常与`压缩`结合一块使用的文件管理任务
- tar, 磁带打包工具(普通归档.tar/压缩归档.tgz)
    - tar -czvf xxx
    - tar -xzvf xxx
- zip, 打包并压缩文件
- rsync, 同步远端文件和目录

# Reference
0. [快乐的Linux命令行](http://billie66.github.io/TLCL/index.html)
0. [Linux命令大全](https://www.runoob.com/linux/linux-command-manual.html)
