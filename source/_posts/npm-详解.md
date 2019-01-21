---
title: npm 详解
date: 2019-01-21 18:01:10
tags: npm
---

nodejs 社区乃至 Web 前端工程化领域发展到今天，作为 node 自带的包管理工具的 npm 已经成为每个前端开发者必备的工具。

## npm 常用命令
### 1.npm init
会在当前文件夹下初始化一个package.json文件，在命令行的提示下填写每一步的信息，当然也可以一直回车，npm会自动给我们填充一些默认值

### 2.npm install
1) npm install // 会自动安装package.json下所有的安装包(依赖)
2) npm install packagename // 安装packagename包
3) npm install packagename@version(例: npm install react@16.0.0) 安装特定版本的packagename包
npm install的时候加入不同的参数有不同的效果
--global/-g 会将依赖加入全局环境中
--save/-S 会将依赖加入package.json的dependencies字段，表示生产环境依赖
--save-dev/-D 会将依赖加入package.json的devDependencies字段，表示开发测试环境依赖
--save-optional/-O 会将依赖加入package.json的optionalDependencies字段，表示可选依赖
```bash
npm install jquery --save-exact 或
npm install jquery -E

如果输入命令为
npm install jquery -ES

留意package.json 文件的 dependencies 字段，以看出版本号中的^消失了
"dependencies": {
    "jquery": "3.2.1"
}
```

### 3.npm cache
其他的没用过，主要用到的是npm cache clean 表示清除npm缓存

### 4.npm config
npm config set registry http://registry.cnpmjs.org // 表示cnpm为npm源镜像
npm config get registry // 查看源镜像

npm info express 表示查看源是否好用
npm –registry http://registry.cnpmjs.org info underscore 表示临时切换underscore的源为http://registry.cnpmjs.org

设置代理 
npm config set proxy http://server:port 
npm config set https-proxy http://server:port

需要认证的代理 
npm config set proxy http://username:password@server:port 
npm confit set https-proxy http://username:password@server:port

淘宝镜像源 
搜索地址：http://npm.taobao.org/ 
registry地址：http://registry.npm.taobao.org/

### 5.其他命令
npm update packagename 更新包
npm uninstall packagename 卸载包(可以添加 --save, --save-dev等参数)
npm outdated packagename 检查包是否过时
npm ls 查看已经安装的模块
npm root 查看包的安装路径
npm view 查看包的注册信息
npm version 查看包的版本

## npm install的实现原理
下面会用一点力气，详细地说npm instal的了解
以npm@5.6.0为例，当你执行npm install的时候会经理以下几个阶段
### 1.执行工程自身的preinstall(前提是当前npm项目定义了preinstall的钩子)
### 2.确定首层依赖模块即dependencies和devDependencies属性中直接指定的模板(前提npm install没有指定安装的模块)
工程本身是整颗依赖树的根节点，每个首层依赖模块都是根节点下面的一棵子树，npm会开启多进程从每个首层依赖模块中逐步寻找更深层次的节点。
### 3.获取模块
获取模块是一个递归的过程，分为以下几个阶段
1) 获取模块信息。在下载一个模块之前，首选要确定器版本 这就是因为package.json中往往是semantic version .此时如果版本描述文件（npm-shrinkwrap.json 或 package-lock.json）中有该模块信息直接拿即可，如果没有则从仓库获取。如pachage.json中某一个包的版本是^1.1.0，npm就会去仓库中获取符合1.x.x 形式的最新版本，
2) 获取模块实体，上一步会获取到模块的压缩包地址（resolved 字段），npm会用此地址检查本地缓存，缓存中有就直接拿，如果没有则从仓库中下载。
3) 查找该模块依赖，如果有依赖则回到第一步，如果没有则停止。
### 4.模块扁平化(dedupe)
上一步获取到的是一颗完整的依赖树，其中可能包含大量重复模块。比如A模块依赖于lodash,B模块同样依赖于lodash.在 npm3 以前会严格按照依赖树的结构进行安装，因此会造成模块冗余。
从 npm3 开始默认加入了一个 dedupe 的过程。它会遍历所有节点，逐个将模块放在根节点下面，也就是 node-modules 的第一层。当发现有重复模块时，则将其丢弃。比如 node-modules 下已经有了一个 lodash@1.0.0，此时又发现某模块下有一个 loader@1.0.0，则直接将其从依赖树中丢弃。
这里需要对重复模块进行一个定义， 它指的是模块名相同的semver兼职，每一个semver都对应一段版本的允许范围，如果两个模块的版本允许范围存在交集，那么就可以得到兼容一个版本，而不必要版本号完全一致，这可以使更多冗余模块在 dedupe 过程中被去掉。

### 5.安装模块
这一步将会更新工程中的node_modules,并执行模块中的生命周期函数（按照 preinstall、install、postinstall 的顺序）。

### 6.执行工程自身生命周期
当前npm工程如果定义了钩子此时会被执行，（按照 preinstall、install、postinstall 的顺序）。
最后一步是生成或者 更新版本描述文件，npm install 过程完成。


最后说下如何定义个npm包的入口执行文件
*** package.json中的main字段定义了该项目或者npm包的可执行入口文件的路径 ***
```json
"main": "lib/ReactPlayer.js"
```