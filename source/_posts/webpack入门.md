---
title: webpack4.x入门配置 && 优化打包速度(持续更新！！！)
date: 2018-09-04 13:16:42
tags: webpack
---
### 概念
官方解释：本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。
个人理解：webpack作为一个现代前端打包工具，他的出现顺应了前端工程 *** 组件化 *** *** 模块化 *** 的趋势，它的出现改善了前端开发生态，虽然之前也出现过一些处理模块依赖的工具或者插件但其中没有一个拥有webpack这么全面的能力，当然很多能力都得益于webpack插件的实现，但这也不可否认webpack本身出现的重要意义。

### webpack 配置(webpack.config.js)

```javascript
// config.js
'use strict'

const path = require('path')

module.exports = {
  dev: {

    // Paths
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {},

    host: 'localhost',
    port: 3001,
    autoOpenBrowser: false,
    errorOverlay: true,
    notifyOnErrors: true,
    poll: false,

    useEslint: false,
    showEslintErrorsInOverlay: false,

    /**
     * Source Maps
     */

    // https://webpack.js.org/configuration/devtool/#development
    devtool: 'cheap-module-eval-source-map',

    cacheBusting: false,

    cssSourceMap: true
  },

  build: {
    // Template for index.html
    index: './src/index.html',

    // Paths
    // 打包文件根目录
    assetsRoot: path.resolve(__dirname, '../dist/app'),
    // 静态资源根目录
    staticRoot: path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    assetsPublicPath: './',

    // 入口paths
    entryIndex: path.resolve(__dirname, '../src/index.js'),
    entryShowStudent: path.resolve(__dirname, '../src/buttonWin/student.js'),
    entryShowTeacher: path.resolve(__dirname, '../src/buttonWin/teacher.js'),
    entryShowNavigation: path.resolve(__dirname, '../src/app/navigation/index.js'),

    /**
     * Source Maps
     */
    productionSourceMap: false,
    // https://webpack.js.org/configuration/devtool/#production
    devtool: '#source-map',

    productionGzip: true,
    productionGzipExtensions: ['js', 'css'],

    bundleAnalyzerReport: process.env.npm_config_report
  }
}

```

```javascript
// utils.js
'use strict'
const path = require('path')
const config = require('./config')
const packageConfig = require('../package.json')

exports.assetsPath = function (_path) {
  const assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsSubDirectory

  return path.posix.join(assetsSubDirectory, _path)
}

exports.createNotifierCallback = () => {
  const notifier = require('node-notifier')

  return (severity, errors) => {
    if (severity !== 'error') return

    const error = errors[0]
    const filename = error.file && error.file.split('!').pop()

    notifier.notify({
      title: packageConfig.name,
      message: severity + ': ' + error.name,
      subtitle: filename || '',
      icon: path.join(__dirname, 'logo.png')
    })
  }
}
```

```javascript
// webapck.base.conf.js
'use strict'
const path = require('path')
const HappyPack = require('happypack');
const os = require('os');
const happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length });
const utils = require('./utils')
const config = require('./config')

function resolve (dir) {
  return path.join(__dirname, '..', dir)
}

module.exports = {
  /**
   * webpack 在寻找相对目录文件时会以context为根目录
   * 默认是执行启动webpack时所在的当前工作目录
   */
  context: path.resolve(__dirname, '../'),
  /**
   * 入口文件(一般是对象如果是多页面应用也可以是数组)
   * 指定webpack开始构建的入口模块，从该模块开始构建并计算出直接或间接依赖的模块或者库
   */ 
  entry: {
    index: './src/index.js'
  },
  /**
   * 出口文件(定义了编译之后的文件存放位置，文件名等信息)
   * 告诉webpack如何命名输出的文件以及输出的目录
   * */ 
  output: {
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  // 配置模块被如何解析
  resolve: {
    // 自动解析确定的扩展，是用户在用户在引用的时候可以不带扩展
    extensions: ['.js', '.json'],
    // 创建 import 或 require 的别名，来确保模块引入变得更简单
    alias: {
      '@': resolve('src')
    }
  },
  // 决定如何处理项目中的不同模块

  /**
   * 由于webpack只能处理javascript，所以我们需要对一些非js文件处理成webpack能够处理的模块，比如sass文件
  */
  module: {
    // 创建模块时的，匹配请求的规则数组这些规则能够修改模块的创建方式。这些规则能够对模块(module)应用 loader，或者修改解析器(parser)。
    rules: [
      { 
        test: /\.css$/,
        use: [
          {
            loader: 'style-loader'
          },
          {
            loader: 'css-loader'
          }
        ]
      },
      {
        test: /\.js$/,
        loader: 'happypack/loader?id=happyBabel',
        exclude: /node_modules/
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ],
    noParse: [/libs/]
  },
  // webpack 插件列表
  plugins: [
    new HappyPack({
      id: 'happyBabel',
      loaders: [{
        loader: 'babel-loader',
        options: {
          cacheDirectory: true
        }
      }],
      threadPool: happyThreadPool,
      verbose: true
    })
  ],
}
```
*** 开发模式 ***
开发模式使用webpack-dev-server起localhost服务

```javascript
'use strict'
const utils = require('./utils')
const webpack = require('webpack')
const config = require('./config')
const merge = require('webpack-merge')
const path = require('path')
const baseWebpackConfig = require('./webpack.base.conf')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
const portfinder = require('portfinder')

const HOST = process.env.HOST
const PORT = process.env.PORT && Number(process.env.PORT)

const devWebpackConfig = merge(baseWebpackConfig, {
  mode: 'development',
  // source-map模式 https://www.webpackjs.com/configuration/devtool/
  devtool: config.dev.devtool,

  // webpack-dev-server 相关配置参数 https://www.webpackjs.com/configuration/dev-server/
  devServer: {
    // 当使用内联模式(inline mode)时，在开发工具(DevTools)的控制台(console)将显示消息，如：在重新加载之前，在一个错误之前，或者模块热替换(Hot Module Replacement)启用时。这可能显得很繁琐。
    clientLogLevel: 'none',
    // 是否热加载
    hot: true,
    /**
     * 告诉服务器从哪里提供内容。只有在你想要提供静态文件时才需要。
     * 这样就不需要使用CopyWebpackPlugin插件转移静态文件了，可以直接基于src目录提供图片等静态内容
     * */ 
    contentBase: path.resolve(__dirname, '../src/'),
    // 是否启用gzip压缩
    compress: true,
    host: HOST || config.dev.host,
    port: PORT || config.dev.port,
    // 是否自动打开浏览器
    open: config.dev.autoOpenBrowser,
    // 当存在编译器错误或警告时，在浏览器中显示错误或者警告并全屏覆盖
    overlay: config.dev.errorOverlay
      ? { warnings: false, errors: true }
      : false,
    /**
     * 此路径下的打包文件可在浏览器中访问。
     * 用于确定应该从哪里提供 bundle.js
     * 假设服务器运行在 http://localhost:8080 并且 output.filename 被设置为 bundle.js。默认 publicPath 是 "/"，所以你的包(bundle)可以通过 http://localhost:8080/bundle.js 访问。
    */
    publicPath: config.dev.assetsPublicPath,
    // 是否启用代理服务器
    proxy: config.dev.proxyTable,
    // 启用 quiet 后，除了初始启动信息之外的任何内容都不会被打印到控制台
    quiet: true,
    // 是否监视contentBase目录下的文件变动(一般会是一些静态图片资源，如果设置为true在发生图片替换的时候也会出发重新加载或热加载)
    watchContentBase: true,
    watchOptions: {
      poll: config.dev.poll,
      ignored: /node_modules/
    }
  },
  plugins: [
    // 热更新HMR插件(搭配 { hot: true } 使用))
    new webpack.HotModuleReplacementPlugin(),
    // 当开启 HMR 的时候使用该插件会显示模块的相对路径，建议用于开发环境。
    new webpack.NamedModulesPlugin(),
    // 确保在遇到编译错误的时候不会退出进程
    new webpack.NoEmitOnErrorsPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'src/index.html',
      inject: true
    }),
    // 代码分割(webpack4.x废弃了commonChunks插件)
    new webpack.optimize.SplitChunksPlugin({
      chunks: "all",
      minSize: 20000,
      minChunks: 1,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      name: true,
      cacheGroups: {
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10
        }
      }
    })
  ]
})

module.exports = new Promise((resolve, reject) => {
  portfinder.basePort = process.env.PORT || config.dev.port
  portfinder.getPort((err, port) => {
    if (err) {
      reject(err)
    } else {
      // publish the new Port, necessary for e2e tests
      process.env.PORT = port
      // add port to devServer config
      devWebpackConfig.devServer.port = port

      // Add FriendlyErrorsPlugin
      devWebpackConfig.plugins.push(new FriendlyErrorsPlugin({
        compilationSuccessInfo: {
          messages: [`Your application is running here: http://${devWebpackConfig.devServer.host}:${port}`]
        },
        onErrors: config.dev.notifyOnErrors
          ? utils.createNotifierCallback()
          : undefined
      }))

      resolve(devWebpackConfig)
    }
  })
})
```
*** 生产环境 ***

```javascript
// webpack.prod.conf.js
'use strict'

const path = require('path')
const utils = require('./utils')
const webpack = require('webpack')
const config = require('./config')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
const CopyWebpackPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

const ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin')

const webpackConfig = merge(baseWebpackConfig, {
  mode: 'production',
  devtool: config.build.productionSourceMap ? config.build.devtool : false,
  entry: {
    index: config.build.entryIndex,
    showStudent: config.build.entryShowStudent,
    showTeacher: config.build.entryShowTeacher,
    showNavigation: config.build.entryShowNavigation
  },
  output: {
    path: config.build.assetsRoot,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': JSON.stringify('production')
    }),
    // ParallelUglifyPlugin 插件可以多进程平行压缩混淆js代码，会比原生的webpack uglify插件打包速度快很多
    new ParallelUglifyPlugin({
      uglifyJS: {
        output: {
          comments: false
        },
        compress: {
          warnings: false
        }
      },
      sourceMap: config.build.productionSourceMap
    }),
    // webpack4.x采用MiniCssExtractPlugin代替了ExtractTextPlugin插件来抽离css代码
    new MiniCssExtractPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css'),
      allChunks: true
    }),
    // 压缩并去重css文件
    new OptimizeCSSPlugin({
      cssProcessorOptions: config.build.productionSourceMap
        ? { safe: true, map: { inline: false } }
        : { safe: true }
    }),
    // 生成html并引入js/css文件
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: config.build.index,
      inject: true,
      chunks: ['index'],
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      }
    }),
    new HtmlWebpackPlugin({
      filename: 'showTeacher.html',
      template: config.build.index,
      inject: true,
      chunks: ['showTeacher'],
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      }
    }),
    new HtmlWebpackPlugin({
      filename: 'showStudent.html',
      template: config.build.index,
      inject: true,
      chunks: ['showStudent'],
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      }
    }),
    new HtmlWebpackPlugin({
      filename: 'showNavigation.html',
      template: config.build.index,
      inject: true,
      chunks: ['showNavigation'],
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
      }
    }),
    // keep module.id stable when vendor modules does not change
    new webpack.HashedModuleIdsPlugin(),
    // enable scope hoisting
    new webpack.optimize.ModuleConcatenationPlugin(),

    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../src/libs'),
        to: path.resolve(config.build.staticRoot, 'libs'),
        ignore: ['.*']
      },
      {
        from: path.resolve(__dirname, '../src/dragons'),
        to: path.resolve(config.build.staticRoot, 'dragons'),
        ignore: ['.*']
      },
      {
        from: path.resolve(__dirname, '../src/music'),
        to: path.resolve(config.build.staticRoot, 'music'),
        ignore: ['.*']
      },
      {
        from: path.resolve(__dirname, '../src/phaser-img'),
        to: path.resolve(config.build.staticRoot, 'phaser-img'),
        ignore: ['.*']
      },
      {
        from: path.resolve(__dirname, '../src/video'),
        to: path.resolve(config.build.staticRoot, 'video'),
        ignore: ['.*']
      },
      {
        from: path.resolve(__dirname, '../src/fonts'),
        to: path.resolve(config.build.staticRoot, 'fonts'),
        ignore: ['.*']
      }
    ])
  ]
})

if (config.build.productionGzip) {
  const CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}

if (config.build.bundleAnalyzerReport) {
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig

```

```javascript
// build.js
'use strict'

process.env.NODE_ENV = 'production'

const ora = require('ora')
const rm = require('rimraf')
const path = require('path')
const chalk = require('chalk')
const webpack = require('webpack')
const config = require('./config')
const webpackConfig = require('./webpack.prod.conf')

const spinner = ora('building for production...')
spinner.start()

rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
  webpack(webpackConfig, (err, stats) => {
    spinner.stop()
    if (err) throw err
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false, // If you are using ts-loader, setting this to true will make TypeScript errors show up during build.
      chunks: false,
      chunkModules: false
    }) + '\n\n')

    if (stats.hasErrors()) {
      console.log(chalk.red('  Build failed with errors.\n'))
      process.exit(1)
    }

    console.log(chalk.cyan('  Build complete.\n'))
  })
})
```

```json
// package.json
{
  "scripts": {
    "dev": "NODE_ENV=development webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "build": "node build/build.js"
  },
}
```

webpack 迁移到4.x其实变动不大，主要有两点
1.相关loader需要升级
2.相关插件需要升级还有部分插件被废弃

增加了一个mode参数来区分开发跟生产环境

优化的思路主要使用HappyPack多进程babel编译插件加快编译速度，使用ParallelUglifyPlugin代替原有的UglifyJsPlugin，同样是可以多进程执行js代码的压缩混淆


参考文档
[webpack4.x更新日志(翻译版)](https://juejin.im/post/5a951bf851882524d842ec8b)
[webpack性能优化](https://www.cnblogs.com/imwtr/p/7801973.html)
[webpack4升级完全指南](https://segmentfault.com/a/1190000014247030)
[Webpack 3.X - 4.X 升级记录](https://blog.csdn.net/qq_16559905/article/details/79404173)