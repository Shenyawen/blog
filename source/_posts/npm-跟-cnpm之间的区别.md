---
title: npm 跟 cnpm之间的区别
date: 2018-12-27 16:14:40
tags: javascript
---
npm 一款基于nodeJs的包管理工具，也是目前Node社区最流行且支持第三方模块最多的包管理器

## package.json
1.基本上每个js项目根目录下都有一个package.json文件，定义了该项目所需要的各种模块依赖，以及项目的配置信息(比如名称/版本/许可证等元数据). npm install 会根据这个配置文件自动下载所需木块即配置项目所需的运行和开发环境
2.package.json 可以手动编写，也可以使用npm init命令自动生成, 两者并没有什么区别


## package-lock.json
package.json版本号前面的'^'表示向后兼容依赖，即在大版本不变的情况下，下载最新包，每次在npm i之后下载的包都会发生变化，为了系统的稳定性考虑，每次执行完npm install之后会对应生成package-lock文件，用以记录当前状态下实际安装的各个npm package的具体来源跟版本号，相当于是提供了一个参考，在出现版本兼容性问题的时候，就可以参考这个文件来修改版本号即可。

注意：这两天遇到一个坑，npm i一直失败，报找不到文件夹的错误，刚开始以为是node_modules的错误，就把整个node_nodules都删掉了，但依然不行。最后看到了package-lock.json这货，mmp想着可能就是这BK的锅，果断给它删了，然后npm i就行了.... what fuck(可能是因为之前cnpm跟npm混着用造成的后果)
{% asset_img error.jpg 目标图片 %}

## npm 跟 cnpm的区别
cnpm 是淘宝团队提供的一个npm国内替代工具，但是一定有点需要注意就是不要cnpm 跟 npm不要混用，虽然官网上说两者表现一致，但内部实现还是有点区别的，最好的方式应该是使用npm但是切换成淘宝镜像
```bash
npm set registry https://registry.npm.taobao.org
```

有关npm的更多内容可以去npm中文官网看 https://www.npmjs.cn/

必要的时候可以选择切换npm到yarn, 体验上会比npm好很多(*** 脸书出品，必属精品！***)