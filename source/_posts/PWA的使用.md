---
title: PWA的使用
date: 2019-01-29 10:57:06
tags: H5
---

PWA => Progressive web application 渐进式web应用

## PWA 是什么？
Progressive Web Apps (下文以“PWAs”代指) 是一个令人兴奋的前端技术的革新。PWAs综合了一系列技术使你的 web app表现得就像是 native mobile app。相比于纯 web 解决方案和纯 native 解决方案，PWAs对于开发者和用户有以下优点：

1.你只需要基于开放的 W3C 标准的 web 开发技术来开发一个app。不需要多客户端开发。

2.用户可以在安装前就体验你的 app。

3.不需要通过 AppStore 下载 app。app 会自动升级不需要用户升级。

4.用户会受到‘安装’的提示，点击安装会增加一个图标到用户首屏。

5.被打开时，PWA 会展示一个有吸引力的闪屏。

6.chrome 提供了可选选项，可以使 PWA 得到全屏体验。

7.必要的文件会被本地缓存，因此会比标准的web app 响应更快（也许也会比native app响应快）

8.安装及其轻量 -- 或许会有几百 kb 的缓存数据。

9.网站的数据传输必须是 https 连接。

10.PWAs 可以离线工作，并且在网络恢复时可以同步最新数据。

PWA 技术目前被 Firefox，Chrome 和其他基于Blink内核的浏览器支持。微软正在努力在Edge浏览器上实现。唯一不幸的是麻蛋苹果不支持。

但即使苹果不支持PWA, 但依然不影响我们将网站改造成PWA，因为它是渐进式的，就算离线访问不能实现，但其他功能都像原来一样没有影响。

## 如果实现一个简单的PWA
PWA的核心是service-worker(SW), 任何PWA都有一个service-worker.js(sw.js)文件, 目前大部分前端项目都深度集成了webpack, 在webpack打包项目中使用service-worker会有以下两个比较严重的问题
1.webpack生成的资源一般都会带有一串hash, sw的资源列表里面需要同步更新这些带hash的资源
2、每次更新代码，都需要通过更新sw文件版本号来通知客户端对所缓存的资源进行更新。（其实只要这一次的sw代码和上一次的sw代码不一样即可触发更新，但使用明确的版本号会更加合适）。

所幸现在webpack社区已经有人做了这件事，除了官方推荐的[sw-precache-webpack-plugin](https://github.com/goldhand/sw-precache-webpack-plugin), 还有一个就是我现在用的[offline-plugin](https://github.com/NekR/offline-plugin), 相比sw-precache-webpack-plugin，offline-plugin可能会有一下优点

1.更多的可选配置项，满足更加细致的配置要求；
2.更为详细的文档和例子；
3.更新频率相对更高，star数更多；
4.自动处理生命周期，用户无需纠结生命周期的坑；
5.支持AppCache；

### 基本使用
*** 安装 ***
```bash
npm install offline-plugin [--save-dev]
```
*** 使用步骤 ***
第一步：在webpack中使用
webpack-dev-server是将依赖的资源存入内存，不能跟SW搭配使用，只有在打完包之后使用，我是在本地打完包再在dist文件夹起一个server服务来测试SW有没有安装成功的
```javascript
// webpack.prod.js
const OfflinePlugin = require('offline-plugin')
module.exports = {
  ...,
  plugins: [
    new OfflinePlugin({
      responseStrategy: 'cache-first',
      AppCache: false,
      safeToUseOptionalCaches: true,
      autoUpdate: true,
      caches: {
        main: [
          '**/*.js',
          '**/*.css',
          /\.(png|jpe?g|gif|svg)(\?.*)?$/,
          '/'
        ],
        additional: [
        ]
      },
      externals: [],
      excludes: ['**/.*', '**/*.map', '**/*.gz', '**/manifest-last.json'],
      ServiceWorker: {
        output: './sw.js',
        publicPath: `./sw.js`,
        scope: '/',
        minify: true,
        events: true
      }
    })
  ],
  ...
}
```
第二步：将runtime添加到项目入口文件中
```javascript
require('offline-plugin/runtime').install()

// ES6/Babel/TypeScript
import * as OfflinePluginRuntime from 'offline-plugin/runtime'
OfflinePluginRuntime.install()
```

经过上面的步骤，offline-plugin已经集成到项目之中，直接使用webpack构建即可，但是这样还不可以，你肯定需要一定的自定义配置

### 配置
offline-plugin 本身提供了丰富的参数选项，下面我会说下我在项目中用到的
首先是在webpack中使用
```javascript
new OfflinePlugin({
  responseStrategy: 'cache-first', // 这句话表示缓存优先
  AppCache: false, // 是否使用AppCache， 虽然AppCache已经被W3C废弃，但是依然有部分浏览器支持，所以offline-plugin默认支持AppCache
  safeToUseOptionalCaches: true, // Removes warning for about `additional` section usage
  autoUpdate: true, // 自动更新
  caches: { // webpack 打包正则匹配
    main: [
      '**/*.js',
      '**/*.css',
      /\.(png|jpe?g|gif|svg)(\?.*)?$/,
      '/'
    ],
    additional: [],
    optional: []
  },
  externals: [], // 需要缓存的外部链接，例如配置http://hello.com/getuser，那么在请求这个接口的时候就会进行接口缓存
  excludes: ['**/.*', '**/*.map', '**/*.gz', '**/manifest-last.json'], // 需要过滤的文件
  ServiceWorker: {
    output: './sw.js', // 输出目录
    publicPath: './sw.js', // sw.js加载路劲
    scope: '/', // 作用域
    minify: true, // 开启压缩
    events: true // 当sw状态改变时候发射对应事件
  }
})

```

着重介绍main 跟 additional字段表示的意义
main: [] 这里配置的是serviceWorker在install阶段需要缓存的文件清单，如果其中有一个失败了，那么整个serviceWorder就会安装失败，所以必须谨慎配置
additional: [] 这里配置的文件清单会在serviceWorker activate的时候进行缓存，与main不一样，如果这里的文件缓存失败，不会影响serviceWorker的正常安装。而且，在接下来页面的ajax异步请求中，还能进行缓存尝试
optional: [] 这里配置的文件清单在serviceWorker安装激活阶段不会进行缓存，只有在监听到网络请求的时候才进行缓存。
刚开始写的时候将静态资源放在additional中进行缓存，总是不能离线访问，直到我把静态资源放到main中缓存之后，居然就可以实现离线访问了，具体原因我也是不太清楚
还要一个需要注意的坑是，有些网络教程是scope设置成/static/sw.js, 最后在网页中会提示，sw最大的作用域权限在/static下面，言外之意这么写是无法将sw的作用域设置在/根路径下面。那就不能缓存html文件了, 然后还整了一堆有的没有的解决办法，耽误挺长时间的，其实直接将sw.js打到根目录，然后设置scope为/就行了。

在入口文件中使用
```javascript
OfflinePluginRuntime.install({
  // 监听sw事件，当更新ready的时候，调用applyUpdate以跳过等待，新的sw立即接替老的sw
  onUpdateReady: () => {
    console.log('SW Event:', 'onUpdateReady')
    OfflinePluginRuntime.applyUpdate()
  },
  onUpdated: () => {
    console.log('SW Event:', 'onUpdated')
    window.swUpdate = true
  }
})
```
### 降级处理
```javascript
if ('serviceWorker' in navigator) {
  console.log('Start trying Registrating Scope')
  navigator.serviceWorker.register('sw.js', { scope: '/' })
    .then((reg) => {
      // registration worked
      console.log('Registration succeeded. Scope is ' + reg.scope)
    })
    .catch((error) => {
      // registration failed
      console.log('Registration failed with ' + error)
    })
}
```

PWA其实跟manifest.json没有关系，有它还是没他都不影响service-worker的工作，它的作用告诉浏览器当把页面发送到主屏幕上时该怎么显示: 应用名称 应用图标 启动页面....
还有manifest.json在苹果浏览器中并不能正常工作，但是可以用下面这个设置模拟实现
```html
<!-- 应用图标： -->
<link rel="apple-touch-icon" href=“/custom_icon.png">

<!-- 启动画面： -->
<link rel="apple-touch-startup-image" href="/launch.png">

<!-- 应用名称： -->
<meta name="apple-mobile-web-app-title" content="AppTitle">

<!-- 全屏效果： -->
<meta name="apple-mobile-web-app-capable" content="yes">

<!-- 设置状态栏颜色： -->
<meta name="apple-mobile-web-app-status-bar-style" content="black">
```
如果各位还想继续深入了解PWA内部实现原理可以参考下[借助Service Worker和cacheStorage缓存及离线开发](https://www.zhangxinxu.com/wordpress/2017/07/service-worker-cachestorage-offline-develop/)、[CacheStorage Api](https://developer.mozilla.org/zh-CN/docs/Web/API/CacheStorage)