---
title: http https http2介绍
date: 2019-02-13 10:21:16
tags: http
---

我们绝大多数的Web应用都是基于HTTP来进行开发的。我们对Web的操作都是通过HTTP协议来进行传输数据的。简单来说就是HTTP协议就是客户端和服务器交互的一种通讯格式。
可以说，Http就是Web通信的基础，这是我们必学的。

到现在位置， HTTP协议已经有三个版本了, http0.9太老就不说了
HTTP1.0
HTTP1.1
HTTP/2

## HTTP版本之间的区别
### HTTP1.0和HTTP1.1区别

HTTP1.1与HTTP1.0最主要的区别就是1.1支持持久化链接，而1.0默认是短链接即每次与服务器交互，都需要新开一个连接！
试想一下：请求一张图片，新开一个连接，请求一个CSS文件，新开一个连接，请求一个JS文件，新开一个连接。HTTP协议是基于TCP的，TCP每次都要经过三次握手，四次挥手，慢启动…这都需要去消耗我们非常多的资源的！

在HTTP1.1中默认就使用持久化连接来解决：建立一次连接，多次请求均由这个连接完成！(如果阻塞了，还是会开新的TCP连接的)

相对于持久化连接还有另外比较重要的改动：

HTTP 1.1增加host字段

HTTP 1.1中引入了Chunked transfer-coding，范围请求，实现断点续传(实际上就是利用HTTP消息头使用分块传输编码，将实体主体分块传输)

HTTP 1.1管线化(pipelining)理论，客户端可以同时发出多个HTTP请求，而不用一个个等待响应之后再请求

注意：这个pipelining仅仅是限于理论场景下，大部分桌面浏览器仍然会选择默认关闭HTTP pipelining！

所以现在使用HTTP1.1协议的应用，都是有可能会开多个TCP连接的！

### HTTP2基础
上面也已经说了，HTTP 1.1提出了管线化(pipelining)理论，但是仅仅是限于理论的阶段上，这个功能默认还是关闭了的。
![111](https://mmbiz.qpic.cn/mmbiz_jpg/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemhVrib5l2daeFVQAEIVWK2yGn1UzvEfYYXXWCGxXib6pHHVaGCD2AFmMw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)
![1111](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemrmZuVZMTAl5zicJukuywvFYZMiavyw0LBkziaq5FbbbicWLeLRPOXUvu4g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTP Pipelining其实是把多个HTTP请求放到一个TCP连接中一一发送，而在发送过程中不需要等待服务器对前一个请求的响应；只不过，客户端还是要按照发送请求的顺序来接收响应！

就像在超市收银台或者银行柜台排队时一样，你并不知道前面的顾客是干脆利索的还是会跟收银员/柜员磨蹭到世界末日（不管怎么说，服务器（即收银员/柜员）是要按照顺序处理请求的，如果前一个请求非常耗时（顾客磨蹭），那么后续请求都会受到影响。

在HTTP1.0中，发送一次请求时，需要等待服务端响应了才可以继续发送请求。

在HTTP1.1中，发送一次请求时，不需要等待服务端响应了就可以发送请求了，但是回送数据给客户端的时候，客户端还是需要按照响应的顺序来一一接收

所以说，无论是HTTP1.0还是HTTP1.1提出了Pipelining理论，还是会出现阻塞的情况。从专业的名词上说这种情况，叫做线头阻塞（Head of line blocking）简称：HOLB

### HTTP1.1和HTTP2区别
HTTP2与HTTP1.1最重要的区别就是解决了 *** 线头阻塞 *** 的问题！其中最重要的改动是：*** 多路复用 (Multiplexing) ***

多路复用意味着线头阻塞将不在是一个问题，允许同时通过单一的 HTTP/2 连接发起多重的请求-响应消息，合并多个请求为一个的优化将不再适用。

(我们知道：HTTP1.1中的Pipelining是没有付诸于实际的)，之前为了减少HTTP请求，有很多操作将多个请求合并，比如：Spriting(多个图片合成一个图片)，内联Inlining(将图片的原始数据嵌入在CSS文件里面的URL里），拼接Concatenation(一个请求就将其下载完多个JS文件)，分片Sharding(将请求分配到各个主机上)……

![2222](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemqUVhTDUO0shiasqkkwib4eIAia6U6VX4EVFp4ZzbL1JPCcekQlNHwoKicQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)


HTTP2所有性能增强的核心在于 *** 新的二进制分帧层 *** (不再以文本格式来传输了)，它定义了如何封装http消息并在客户端与服务器之间传输。
![3333](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemBz50EHfzfXHje06D4ialibWpK3bY8NOjYu0Wib9jFrqjMegtpGzufUK6Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

看上去协议的格式和HTTP1.x完全不同了，实际上HTTP2并没有改变HTTP1.x的语义，只是把原来HTTP1.x的header和body部分用frame重新封装了一层而已
![333](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemgYXqL5rTvMicTxd8wUSiaTAoH6Soj6Za0cXZoQXcuZvvibgdLTCRHVv6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTP2连接上传输的每个帧都关联到一个“流”。流是一个独立的，双向的帧序列可以通过一个HTTP2的连接在服务端与客户端之间不断的交换数据。
![444](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeembxke1sr7QjDiaaAl836pEscScKrReJ5KaTJoQISeqHdlh31lmcUBdEg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

实际上运输时：
![555](https://mmbiz.qpic.cn/mmbiz_png/2BGWl1qPxib2fuoQwW0ugqdD8wxEWXeemdeqGb0YSr9qcdibzDJPPshSYJMl92kAGc6UGibDNGYmYHkXRKOibmE1Tw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

HTTP2还有一些比较重要的改动：

使用HPACK对HTTP/2头部压缩

服务器推送

  HTTP2推送资料：https://segmentfault.com/a/1190000015773338

流量控制

  针对传输中的流进行控制(TCP默认的粒度是针对连接)

流优先级（Stream Priority）它被用来告诉对端哪个流更重要。


有效的证书需要由权威机构CA签名，CA会用自己的私钥来生成数字签名。这个权威机构CA客户端是可以完全信任的，客户端浏览器会安装CA的根证书，由CA签名的证书是被CA所信任的，这就构成了信任链，所以客户端可以信任该服务器的证书。 
客户端与服务器建立ssl连接时，服务器将自身的证书传输给客户端，客户端在验证证书的时候，先看CA的根证书是否在自己的信任根证书列表中。再用CA的根证书提供的公钥来验证服务器证书中的数字签名，如果公钥可以解开签名，证明该证书确实被CA所信任。再看证书是否过期，访问的网站域名与证书绑定的域名是否一致。这些都通过，说明证书可以信任。

接下来使用服务器证书里面的公钥进行服务器身份的验证。 客户端生成一个随机数给到服务器。 服务器对随机数进行签名（加密），并回传给到客户端。 客户端用服务器证书的公钥对随机数的签名进行验证，若验证通过，则说明对应的服务器确实拥有对应服务器证书的私钥，因此判断服务器的身份正常。否则，则任务服务器身份被伪造。这些都没问题才说明服务器是可信的。

接下来客户端会生成会话密钥，使用服务器公钥加密。服务器用自己的私钥解密后，用会话密钥加密数据进行传输。ssl连接就建立了。

参考文档：https://mp.weixin.qq.com/s/bobcUDUg9nXlJatLiVaaoQ
        https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483733&idx=2&sn=93b359af4397cd4afa791fdb5f51f0b5&chksm=ebd74054dca0c942be5180cdf0f460ed7f534ca51230147fe0081df15adac76dac9e61d97761&scene=21#wechat_redirect
        https://mp.weixin.qq.com/s/adZC0N5Fd4X9FjxUrdlS1w
        https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650825181&idx=1&sn=62bb9652c0236e4b0a9fe4848981493e&chksm=80b7b543b7c03c55e5a86416c3523bdba598456fba9dc5597d5ccce324db43c80d8037e2d68f&scene=21#wechat_redirect
        https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483733&idx=1&sn=f9ab8d07d2151bd40cdcd9a290317346&chksm=ebd74054dca0c942a36e6e63c783e9b1f414a16e2c702ae4b371a204960a50c7ae89af207139&scene=21#wechat_redirect
        http://blog.51cto.com/wwdhks/2137261
        https://blog.csdn.net/zwjemperor/article/details/80719427