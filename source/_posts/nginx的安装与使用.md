---
title: nginx的安装与使用
date: 2018-09-23 11:50:58
tags: nginx
---

环境linux

### 安装前准备
因为nginx的一些模块需要依赖一些lib库，所以在安装nginx之前必须先安装这些lib库，这些依赖库主要有g++、gcc、openssl-devel、pcre-devel和zlib-devel 所以执行如下命令安装
```bash
[root@yeshengsheji ~]# yum install gcc-c++
[root@yeshengsheji ~]# yum install pcre pcre-devel
[root@yeshengsheji ~]# yum install zlib zlib-devel
[root@yeshengsheji ~]# yum install openssl openssl--devel
```

### 安装nginx
安装之前首先需要检测一下是否已经安装有nginx
```bash
[root@yeshengsheji ~]# find -name nginx
```
如果系统已经安装nginx则先卸载
```bash
[root@yeshengsheji ~]# yum remove nginx
```
现在可以进入安装步骤了
进入/usr/local目录并选择一个安装包(这里选择1.7.4版本)下载，下载完成之后解压nginx安装包
```bash
[root@yeshengsheji ~]# cd /usr/local
[root@yeshengsheji ~]# wget http://nginx.org/download/nginx-1.7.4.tar.gz
[root@yeshengsheji ~]# tar -zxvf nginx-1.7.4.tar.gz
```
解压之后会产生一个nginx-1.7.4文件目录，进入该目录
```bash
[root@yeshengsheji local]# cd nginx-1.7.4
```
接下来就可以安装了
使用--prefix参数来指定nginx安装目录，如果想使用https服务还需要--with-http_stub_status_module和--with-http_ssl_module加载相应模块，否则只需要指定安装目录即可
```bash
[root@yeshengsheji nginx-1.7.4]# ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
[root@yeshengsheji nginx-1.7.4]# make
[root@yeshengsheji nginx-1.7.4]# make install
```
如果没有报错的话，查看下安装目录
```bash
[root@yeshengsheji ~]# whereis nginx
```
进入安装目录便可以启动或者关闭它了，安装过程至此结束

### nginx使用
1.*** 启动 ***
命令格式：nginx安装目录地址 -c nginx配置文件地址
```bash
[root@yeshengsheji ~]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```

2.*** 停止 ***
一般情况下如果只是更改nginx配置文件的话不需要停止nginx服务，加载新模块的时候就需要停止nginx服务了
有三种方式
*** · 从容停止 · ***
```bash
[root@yeshengsheji ~]# ps -ef|grep nginx // 查看进程号
[root@yeshengsheji ~]# kill -QUIT 2072 // 2072就是那个进程号
```

*** · 快速停止 · ***
```bash
[root@yeshengsheji ~]# ps -ef|grep nginx // 查看进程号
[root@yeshengsheji ~]# kill -TERM 2072 // 2072就是那个进程号
或者
[root@yeshengsheji ~]# kill -INT 2072
```

*** · 强制停止 · ***
```bash
[root@yeshengsheji ~]# pkill -9 nginx
```

3.*** 重启 ***
```bash
[root@yeshengsheji ~]# /usr/local/nginx/sbin/nginx -s reload
```
但是这样之后每次更改nginx配置之后重启很麻烦，可以将nginx命令加入环境变量中
```bash
[root@yeshengsheji ~]# vim /etc/profile
// 在最后加入这两行代码
PATH=$PATH:/usr/local/nginx/sbin
export PATH
```
更改之后使用, 使其生效
```bash
[root@yeshengsheji ~]# source /etc/profile
```
然后可以通过echo $PATH查看是否生效
```bash
[root@yeshengsheji ~]# echo $PATH
```
之后就可以愉快的用 nginx -s reload来操作你的nginx了，不要太爽。。。