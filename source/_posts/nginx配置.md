---
title: nginx配置
date: 2018-09-23 13:32:12
tags: nginx
---

nginx.conf
安装完nginx之后肯定是要自定义一些相关配置才能使用
### nginx配置详解
首先找到nginx的安装目录, 进入配置文件目录
```bash
[root@yeshengsheji ~]# whereis nginx
nginx: /usr/local/nginx
[root@yeshengsheji ~]# cd /usr/local/nginx/conf/
[root@yeshengsheji conf]# vim nginx.conf
#字段表明了Nginx服务是由哪个用户哪个群组来负责维护进程的，默认是nobody
# 查看当前用户命令： whoami
# 查看当前用户所属组命令： groups ，当前用户可能有多个所属组，选第一个即可
user root;
# worker_processes字段表示Nginx服务占用的内核数量
# 为了充分利用服务器性能你可以直接写你本机最高内核
worker_processes  1;

# error_log字段表示Nginx错误日志记录的位置
# 项目多建议在server里单独配置
# 模式选择：debug/info/notice/warn/error/crit
# 上面模式从左到右记录的信息从最详细到最少
# error_log  logs/error.log;
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;
# Nginx执行的进程id,默认配置文件是注释了
# 如果上面worker_processes的数量大于1那Nginx就会启动多个进程
# 而发信号的时候需要知道要向哪个进程发信息，不同进程有不同的pid，所以写进文件发信号比较简单
# 你只需要手动创建，比如下面的位置： touch /usr/local/var/run/nginx.pid
#pid        logs/nginx.pid;

#工作模式及连接数上限
events {
  # 每一个worker进程能并发处理的最大连接数
  # 当作为反向代理服务器，计算公式为： `worker_processes * worker_connections / 4`
  # 当作为HTTP服务器时，公式是除以2
  worker_connections  1024;
}

http {
  # 设定mime类型,类型由mime.type文件定义
  include       mime.types;
  default_type  application/octet-stream;
  # 设定日志格式
  #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
  #                  '$status $body_bytes_sent "$http_referer" '
  #                  '"$http_user_agent" "$http_x_forwarded_for"';
  # 关闭access_log可以让读取磁盘IO操作更快
  #access_log  logs/access.log  main;
  
  sendfile        on;
  # 在一个数据包里发送所有头文件，而不是一个接一个的发送
  #tcp_nopush     on;
  # 不要缓存
  #tcp_nodelay on;
  # 连接超时时间
  # keepalive_timeout  65;

  # 开启gzip压缩
  # 一般不同项目单独配置, 写在server{}里单独配置
  # gzip  on;
  # gzip_disable "MSIE [1-6].";

  # 设定请求缓冲
  # 一般不同项目单独配置, 写在server{}里单独配置
  # client_header_buffer_size    128k;
  # large_client_header_buffers  4 128k;

  # 将配置文件分离
  # 配置不同站点的时候只需要在/usr/local/etc/nginx/sites-enabled/不断增加新的文件即可
  # include /usr/local/etc/nginx/sites-enabled/*;

  # 可以用server_name来区分不同域名跳转不同的项目页面
  server {
    # 监听端口号
    listen       80;
    # 服务名称 默认是localhost 也可以是www.xxx.com之类的网址，这样就可以在一个服务器内放置多个服务了
    server_name  www.todatay.com; 

    # 编码
    charset utf-8;
    # gzip
    gzip            on;
    # http://blog.csdn.net/jessonlv/article/details/8016284
    # 设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。
    # 默认值是0，不管页面多大都压缩。
    # 建议设置成大于1k的字节数，小于1k可能会越压越大。
    # gzip_min_length 1000;
    # gzip_proxied
    # 语法: gzip_proxied [off|expired|no-cache|no-store|private|no_last_modified|no_etag|auth|any] ...
    # 默认值: gzip_proxied off
    # 作用域: http, server, location
    # Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含"Via"的 header头。
    # off - 关闭所有的代理结果数据的压缩
    # expired - 启用压缩，如果header头中包含 "Expires" 头信息
    # no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
    # no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
    # private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
    # no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
    # no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
    # auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
    # any - 无条件启用压缩
    # gzip_proxied    expired no-cache no-store private auth;
    # 匹配MIME类型进行压缩
    # gzip_types      text/plain application/xml application/json;
    # location 后面的路径可以表示接口地址
    location /api {
      # 前端代码放置的根目录
      # 我这里是放到了/root/www/app_bg项目下
      root    /root/www/app_bg;
      # 首页索引文件名称
      index   index.html index.html;
      # try_files 最核心的功能是可以替代rewrite
      # 按顺序检查文件是否存在，返回第一个找到的文件
      # 如果所有的文件都找不到，会进行一个内部重定向到最后一个参数。
      # 这样可以解决一个问题就是通过vue或者react打包的SPA应用在使用h5 history路由的时候刷新404
      try_files $uri $uri/ /index.html;
    }
  }

  server {
    # 监听端口号
    listen       80;
    # 服务名称 默认是localhost 也可以是www.xxx.com之类的网址
    server_name  www.tenmat.com;


    # 编码
    charset utf-8;
    # gzip
    gzip            on;
    # http://blog.csdn.net/jessonlv/article/details/8016284
    # 设置允许压缩的页面最小字节数，页面字节数从header头中的Content-Length中进行获取。
    # 默认值是0，不管页面多大都压缩。
    # 建议设置成大于1k的字节数，小于1k可能会越压越大。
    # gzip_min_length 1000;
    # gzip_proxied
    # 语法: gzip_proxied [off|expired|no-cache|no-store|private|no_last_modified|no_etag|auth|any] ...
    # 默认值: gzip_proxied off
    # 作用域: http, server, location
    # Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含"Via"的 header头。
    # off - 关闭所有的代理结果数据的压缩
    # expired - 启用压缩，如果header头中包含 "Expires" 头信息
    # no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
    # no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
    # private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
    # no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
    # no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
    # auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
    # any - 无条件启用压缩
    # gzip_proxied    expired no-cache no-store private auth;
    # 匹配MIME类型进行压缩
    # gzip_types      text/plain application/xml application/json;

    location / {
      # 前端代码放置的根目录
      # 我这里是放到了/root/www/app_bg项目下
      root    /root/www/app_bg;
      # 首页索引文件名称
      index   index.html index.html;
      # try_files 最核心的功能是可以替代rewrite
      # 按顺序检查文件是否存在，返回第一个找到的文件
      # 如果所有的文件都找不到，会进行一个内部重定向到最后一个参数。
      # 这样可以解决一个问题就是通过vue或者react打包的SPA应用在使用h5 history路由的时候刷新404
      try_files $uri $uri/ /index.html;
    }
  }
  server {
    listen       443 ssl;
    server_name  localhost;

    ssl_certificate      cert/1538323290541.pem;
    ssl_certificate_key  cert/1538323290541.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;

    ssl_ciphers  ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers  on;

    # web https化
    location / {
      root   /root/buguniao/www/buguniao_app;
      index  index.html index.htm;
      try_files $uri $uri/ /index.html;
    }

    # 接口https化
    location /api {
      root   html;
      index  index.html index.htm;
      proxy_pass http://127.0.0.1:8090;
    }
  }
  # server {
    # location的相关配置
    # location @rewrites {
      # 语法：rewrite regex replacement [flag];
      # 默认值：无
      # 作用域：server,location,if
      # 如果一个URI匹配指定的正则表达式regex，URI就按照replacement重写。
      # rewrite按配置文件中出现的顺序执行。flags标志可以停止继续处理。
      # last 停止处理后续rewrite指令集，然后对当前重写的新URI在rewrite指令集上重新查找。
      # break 停止处理后续rewrite指令集，并不在重新查找,但是当前location内剩余非rewrite语句和location外的的非rewrite语句可以执行。
      # redirect 如果replacement不是以http:// 或https://开始，返回302临时重定向
      # permant 返回301永久重定向
      # rewrite ^(.+)$ /index.html last;
    # }
    # 这个是设置缓存策略，把所有的静态文件，设置成在本地永久缓存
    # location~.*\.(gif | jpg | js | css | jpeg | png | bmp | swf) $ {
       # root / xxx/ xxx / xxx / dist;  //这块改成自己dist在的路径
       # add_header Cache - Control max - age = 259200000000;
    # }
    # 设置接口代理，‘/api’开头的请求接口反向代理到其他服务器
    # location / api {
       # rewrite ^ . + api / ? (.*) $ / $1
       # break;
       # include uwsgi_params;
       # proxy_pass http: //japi.juhe.cn;
    # }
    # 设置接口代理，‘/op’开头的请求接口反向代理到其他服务器
    # location / op {
       # rewrite ^ . + op / ? (.*) $ / $1
       # break;
       # include uwsgi_params;
       # proxy_pass http: //op.juhe.cn;
    # }
    # 设置接口代理，‘/happy’开头的请求接口反向代理到其他服务器
    # location / happy {
       # rewrite ^ . + happy / ? (.*) $ / $1
       # break;
       # include uwsgi_params;
       # proxy_pass http: //v.juhe.cn;
    # }
    #对 / 所有做负载均衡+反向代理
    # location / {
       # root   /apps/oaapp;
       # index  index.jsp index.html index.htm;
       # proxy_pass        http://backend;
       # proxy_redirect off;
       # 后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
       # proxy_set_header  Host  $host;
       # proxy_set_header  X-Real-IP  $remote_addr;
       # proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
       # proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
    # }
  # }
  
  # 404页面跳转到404.html，相对于上面的root目录
  # error_page  404              /404.html;
  # 403页面跳转到403.html，相对于上面的root目录
  # error_page  403              /403.html;
  # 50x页面跳转到50x.html
  # error_page   500 502 503 504  /50x.html;
  # location = /50x.html {
  #    root   html;
  } 
}
```

上面的是我的配置文件
主要监听了两个端口 80 跟443
80端口是起了一个静态web服务器
443端口是为了接口https化

配置期间也是遇到了各种问题，好在是一一解决了
下面我简单的说下如何实现的接口https化吧
域名是阿里的，在阿里云的管理后台申请一个免费的ssl证书
审核通过之后会有一个公钥跟私钥文件，同时有一段配置说明
将那两个文件放到 /usr/local/nginx/conf/cert目录下，没有则创建一个cert目录，然后像我上面那样调用就行，带有ssl字段的都是跟https有关的配置，期间发生一个问题，按照阿里云的提示设置完成之后，重启nginx服务居然报错了，原因是安装的时候没有加载ssl相关模块，不用删除现有的nginx,覆盖安装一下就行，可以参考 [安装nginx ssl模块](https://www.cnblogs.com/ghjbk/p/6744131.html)这篇blog

配置完成之后 nginx -t 测试一下还有无错误
没有的话 nginx -s -reload重启下服务 (nginx命令前提是你加入到了环境变量里才能直接用)

重启之后还是没能开始https，可能是防火墙的原因，我的就是，看一下服务器的443端口是否打开
```bash
[root@yeshengsheji ~]# /etc/init.d/iptables status
```
{% asset_img nginx.png 目标图片 %}

没有的话打开一下
```bash
[root@yeshengsheji ~]# vim /etc/sysconfig/iptables
```
按照80端口的形式增加一条记录
然后重启下iptables服务

```bash
[root@yeshengsheji ~]# /etc/init.d/iptables restart
iptables：将链设置为政策 ACCEPT：nat filter                [确定]
iptables：清除防火墙规则：                                 [确定]
iptables：正在卸载模块：                                   [确定]
iptables：应用防火墙规则：                                 [确定]
iptables：载入额外模块：ip_conntrack_netbios_ns ip_conntrac[确定]ip_nat_ftp
```
就ok了，还不明白的话可以参考[linux 查看并对外开放端口（防火墙拦截处理）](https://www.cnblogs.com/blog-yuesheng521/p/7198829.html)

如果是用阿里云的EOS话，因为它使用的是firewalld防火墙，开通https需要把443端口开放，踩了个小坑
当你把所有的nginx配置已经配完之后发现依然不能访问https的时候可以看看firewalld的443端口未开放

下面是firewalld的一些常用命令
```bash
# 查看端口是否开放
firewall-cmd --query-port=80/tcp                ##查询端口号80 是否开启
firewall-cmd --permanent --zone=public --add-port=80/tcp ##永久开放80端口号
firewall-cmd --permanent --zone=public --remove-port=80/tcp ##移除80端口号
## --zone #作用域
## --add-port=80/tcp  #添加端口，格式为：端口/通讯协议
## --permanent   #永久生效，没有此参数重启后失效
systemctl status firewalld.service              ##查看防火墙状态
systemctl [start|stop|restart] firewalld.service  ##启动|关闭|重新启动  防火墙
## 注意：修改设置后，要重启防火墙

# http服务
firewall-cmd --query-service http               ##查看http服务是否支持，返回yes或者no
firewall-cmd --add-service=http                 ##临时开放http服务
firewall-cmd --add-service=http --permanent     ##永久开放http服务
firewall-cmd --reload                           ##重启防火墙生效
# https服务 跟http类似
firewall-cmd --add-service=https --permanent               
firewall-cmd --add-service=https --reload
```