---
title: git中SSH key 生成步骤
date: 2018-08-29 22:21:41
tags: 其他
---

SSH 主要用于远程登录。假定你要以用户名user，登录远程主机host，一条命令就够了
``` bash
$ ssh user@host
```

## 一.SSH key登录
但一般情况远程登录都需要使用密码登录，每次都输入密码就会很麻烦。SSH提供了**公钥登录** 可以省去输入密码的步骤
用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求密码。
但这种方法需要用户提供自己的**公钥**。下面说如何生成自己的公钥跟私钥
## 二.生成SSH key
### 步骤1：设置git的用户名和邮箱（第一次设置的情况）
``` bash
$ git config --global user.name "xxx"
$ git config --global user.email "xxxx@xx.xx"
```
---

### 步骤2：生成秘钥
``` bash
$ ssh-keygen -t rsa -C "xxxx@xx.xx"
```
{% asset_img bash.png 目标图片 %}

运行结束之后会在 **~/.ssh/**目录下生成两个文件：**id_rsa.pub**和**id_rsa**。前者是你的公钥后者是私钥

---

### 步骤3：登录github或者gitlab, 添加ssh即公钥文件 => id_rsa.pub
点击setting进入设置页面，点击new SSH key 按钮将id_rsa.pub里的内容复制粘贴进输入框即可
{% asset_img gitsshkey.png 目标图片 %}

---
## 大功告成！！！