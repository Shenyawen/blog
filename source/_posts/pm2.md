---
title: pm2
date: 2018-10-03 18:35:02
tags: pm2
---

pm2：nodeJs 部署工具

### pm2的优势
一般情况下如果我们在本地开发nodeJS程序一般使用node或者nodemon来启动程序，但是如果用这种方式的话有一个后果就是一旦某段程序报错整个应用可能会崩溃，而pm2启动的node程序则不会，它是node进程的管理工具，可以简化很多node应用管理的繁琐任务，如性能监控、自动重启、负载均衡等等，使用起来也很简单方便。

### pm2使用方法
1.安装pm2
```bash
npm i pm2 -g
```

2.常用命令
*** 启动 ***
参数说明
1) *** --watch ***: 监听应用目录的变化，一发生变化，自动重启。(如果需要精确监听某个/些目录，最好通过配置文件)
2) *** -i --instances ***: 启用多个实例，可用于负载均衡。如果-i 0或者-i max，则根据当前机器核数确定实例数目。
3) *** --ignore-watch ***: 排除监听的目录或者文件，可以是特定文件名，也可以是正则。(*** --ignore-watch="test node_modules "some scripts"" ***)
4) *** -n --name ***: 应用名称。查看应用的时候会用到。
5) *** -o --output <path> ***: 输出标准日志的文件路径。
6) *** -e --error <path> ***: 输出错误日志的文件路径。
[完整的命令参数列表](http://pm2.keymetrics.io/docs/usage/quick-start/#options)

```bash
pm2 start app.js --watch -i 2 # 启动应用
# ======================================== #
pm2 restart app.js # 重启应用
```

*** 停止 ***
停止特定的应用，首先通过*** pm2 list *** 获取应用的名称(--name 指定的名称) 或者进程id
```bash
pm2 stop app_name|app_id
```
如果要停止所用应用，可以
```bash
pm2 stop all
```
*** 查看进程状态 ***
```bash
pm2 list
```

*** 查看某个进程的信息 ***
```bash
pm2 describe 0 # 进程id
```

### 配置文件 ###
自己跑的项目比较简单所以没用过，那就简单说明下
1) 配置文件里的设置项跟命令参数基本一一对应
2) 文件格式json/yaml
3) json格式的配置文件，pm2当作普通的js文件来处理，所以可以在里面添加注释或者编写代码，这对于动态调整配置很有好处
4) 如果启动的时候指定了配置文件，那么命令行参数会被忽略(个别参数除外 --env)
详细配置项可以参考[官方文档](http://pm2.keymetrics.io/docs/usage/pm2-doc-single-page/)

### 环境切换 ###
在实际项目中，我们的应用经常需要在多个环境下部署，比如开发环境、测试环境、生产环境等等。在不同环境下，有时候配置项会有差异，比如链接的数据库地址不同等。

对于这种场景，pm2也可以很好的支持。首先通过在配置文件中通过env_xxx来声明不同环境的配置，然后在启动应用时，通过--env参数指定相应运行环境
*** 环境配置声明 ***

首先，在配置文件中，通过env选项声明多个环境配置。简单说明下：
1) *** env ***为默认的环境配置 (生产环境)，*** env_dev | env_test ***分别是开发及测试环境。可以在配置文件中看到*** NODE_ENV | REMOTE_ADDR ***字段的值是不同的。
2) 在应用中，可以通过*** process.env.REMOTE_ADDR ***等来读取配置中的环境变量
```json
{
  "env": {
    "NODE_ENV": "production",
    "REMOTE_ADDR": "http://www.example.com/"
  },  
  "env_dev": {
    "NODE_ENV": "development",
    "REMOTE_ADDR": "http://wdev.example.com/"
  },
  "env_test": {
    "NODE_ENV": "test",
    "REMOTE_ADDR": "http://wtest.example.com/"
  }
}
```
*** 启动指明环境 ***
假如通过下面启动脚本(开发环境)，那么此时*** process.env.REMOTE_ADDR ***的值就是相应的http://wdev.example.com/
```bash
pm2 start app.js --env dev
```
### 负载均衡 ###
命令如下，表示开启三个进程。例：*** -i 0 ***, 则会根据当前机器核心数自动开启尽可能多的进程
```bash
pm2 start app.js -i 3 # 开启三个进程
pm2 start app.js -i max # 根据机器CPU核数，开启对应数目的进程 
```
[参考文档](http://pm2.keymetrics.io/docs/usage/cluster-mode/#automatic-load-balancing)

### 日志查看 ###
除了可以通过打开日志文件查看日志外，还可以通过*** pm2 logs ***来查看实时日志。这点对于排查线上问题十分重要。
比如某个node服务突然异常重启了，那么可以通过pm2提供的日志工具来查看实时日志，看看是不是脚本出错之类导致的异常重启
```bash
pm2 logs
```
### 命令tab补全 ###
运行*** pm2 --help ***可以看到pm2支持的子命令，那么可以通过下面的方式让pm2命令支持tab自动补全命令
```bash
pm2 completion install
source ~/.bash_profile
```

### 开机自动启动 ###
可以通过*** pm2 startup ***实现开机自动启动。细节可以参考[官方文档](http://pm2.keymetrics.io/docs/usage/startup/), 大致流程如下
1) 通过pm2 save保存当前进程状态
2) 通过pm2 startup [platform]生成开机自启动的命令（记得查看控制台输出）
3) 将步骤2生成的命令，粘贴到控制台进行, 如果没有报错，那么你将收获一个自动重启的应用重启的node应用

### 远程部署 ###
还没有自己尝试，等自己用到的时候在发一篇blog具体说下怎么实现的，可以看一下[官方文档](http://pm2.keymetrics.io/docs/usage/deployment/#getting-started)
现在是通过git的方式，本地更新代码之后再登录到服务器拉取代码重启下服务，但是感觉这样好low逼啊

### 监控 ###
运行一下命令，可以查看当前通过pm2启动的进程状态
```bash
pm2 monit
```

最后说一句, pm2的官方文档写得很好了，基本上只要熟悉常用的几个命令就能把应用跑起来。

