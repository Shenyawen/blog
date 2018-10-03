---
title: H5端唤醒移动客户端程序
date: 2018-10-03 20:19:28
tags: 其他
---

在h5网页中唤起app在很多业务场景中都会有使用，尤其是在一些app推广运营页面中。刚好在之前的开发工作中有用到，好记性不如烂笔头，方便自己之后又遇到了这种场景可以快速实现。

## 唤醒native APP的几种方式 ###
在Android端，常用的方式是Schame + Android Itent，在IOS端，常用的方式是Schema ＋　Universal links（IOS9+）；使用的前提都是客户端程序实现了Schema协议。本文只设计H5端的代码实现。

下面对这3种方式做下简单介绍
### 1.Schema ###
在Android和IOS浏览器中（非微信浏览器），可以通过schema协议的方式唤醒本地app客户端；schema协议在App开发注册之后，与前端进行统一约定，通过H5页面访问某个具体的协议地址，即可打开对应的App客户端 页面；

访问协议地址，目前有三种方式，下面以打开[洋葱数学](https://yangcong345.com/mobileH5V2-ad-pay-page.html)客户端为例：

1) *** 通过a标签打开 ***，点击标签时启动APP
```html
<a href="ycmath://yangcong345.com/userProfile">唤起app</a>
```
2) *** 通过iframe打开 ***，设置iframe.src即会启动
```html
<iframe src="ycmath://yangcong345.com/userProfile"></iframe>
```
3) *** 直接通过window.location 进行跳转 ***
```javascript
window.location.href= "ycmath://yangcong345.com/userProfile";
```
安卓实现注册schema协议，可以参考[Android手机上实现WebApp直接调起NativeApp](https://www.baidufe.com/item/3444ee051f8edb361d12.html)

注：由于微信的白名单限制，无法通过schema来唤起本地app，只有白名单内的app才能通过微信浏览器唤醒，这个问题我目前没有找到合适的解决办法！

### 2.Android Intent ###
在Android Chrome浏览器中，版本号在chrome 25+的版本不再支持通过传统schema的方法唤醒APP，比如通过设置window.location = "xxxx://xxxx"将无法唤醒本地客户端。需要通过Android Intent 来唤醒APP； 使用方式如下：

首先需要构建intent字符串：
```javascript
intent:
login                                                // 特定的schema uri，例如login表示打开NN登陆页
#Intent;
    package=cn.xxxx.xxxxxx;                          // 富途牛牛apk信息
    action=android.intent.action.VIEW;               // 富途牛牛apk信息
    category=android.intent.category.DEFAULT;        // 富途牛牛apk信息
    component=[string];                              // 富途牛牛apk信息,可选
    scheme=xxxx;                                     // 协议类型
    S.browser_fallback_url=[url]                     //可选，schema启动客户端失败时的跳转页，一般为下载页，需通过encodeURIComponent编码
end;
```
然后构造一个a标签，将上面schame 字符串作为其href值，当点击a标签时，即为通过schema打开某客户端登陆页，如果未安装客户端，则会跳转到指定页，这里会跳转到下载页；
```html
<a href="intent://loin#Intent;scheme=ycmath;package=cn.futu.trader;category=android.intent.category.DEFAULT;action=android.intent.action.VIEW;S.browser_fallback_url=http%3A%2F%2Fa.app.qq.com%2Fo%2Fsimple.jsp%3Fpkgname%3Dcn.futu.trader%26g_f%3D991653;end">打开登录页</a>
```

### 3.Universal links ###
Universal links为 iOS 9 上一个所谓 通用链接 的深层链接特性，一种能够方便的通过传统 HTTP 链接来启动 APP, 使用相同的网址打开网站和 APP；通过唯一的网址, 就可以链接一个特定的视图到你的 APP 里面, 不需要特别的 schema；

在IOS中，对比schema的方式，Universal links有以下优点：

通过schema启动app时，浏览器会有弹出确认框提示用户是否打开，而Universal links不会提示，体验更好；
Universal link可在再微信浏览器中打开外部App；

注：网易新闻客户端IOS 9上目前采用这种Universal links方式
针对这部分内容可以参考以下博文
[打通 iOS 9 的通用链接（Universal Links）](http://www.cocoachina.com/ios/20150902/13321.html)
[浏览器中唤起native app || 跳转到应用商城下载（二） 之universal links](https://juejin.im/entry/57bd1e6179bc440063b3a029/view)

### 4.微信JS SDK ###
微信内的网页是一个特殊的存在：对于schema，微信会做出来拦截，导致通过 schema协议无法唤起APP;想要在微信中唤起APP，需要通过微信js sdk提供的接口进行唤起APP,目前微信并未在其开放平台上说明其接口，只有部分在微信白名单（其实就是和腾讯有合作关系的公司）中的应用程序可使用对应的接口进行唤起APP。

QQ webview本身可以通过schema的方式唤起APP，这点还是比较好的

## 实现过程 ##
具体内容可以参考下面这篇博文，文字太多，都是亲测有效的
https://github.com/AlanZhang001/H5CallUpNative
