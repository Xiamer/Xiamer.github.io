---
title: http
date: 2021-04-13 15:37:34
categories: 
- web前端
tags:
- http https
---
## HTTP 和 HTTPS 的区别

### HTTP定义

**HTTP 是一种 <font color=red>超文本传输协议(Hypertext Transfer Protocol)</font>，HTTP 是一个在计算机世界里专门在两点之间传输文字、图片、音频、视频等超文本数据的约定和规范。**

HTTP 主要内容分为三部分，**超文本（Hypertext）、传输（Transfer）、协议（Protocol）。**

* 超文本就是不单单只是本文，它还可以传输图片、音频、视频，甚至点击文字或图片能够进行<font color=red>超链接</font>的跳转。
* 上面这些概念可以统称为数据，传输就是数据需要经过一系列的物理介质从一个端系统传送到另外一个端系统的过程。通常我们把传输数据包的一方称为<font color=red>请求方</font>，把接到二进制数据包的一方称为<font color=red>应答方</font>。
* 而协议指的就是是网络中(包括互联网)传递、管理信息的一些规范。如同人与人之间相互交流是需要遵循一定的规矩一样，计算机之间的相互通信需要共同遵守一定的规则，这些规则就称为协议，只不过是网络协议。
  ![](/images/http/1.jpg)

### HTTPS定义

而 HTTPS 的全称是 <font color=red>Hypertext Transfer Protocol Secure</font>，从名称我们可以看出 HTTPS 要比 HTTPS 多了 secure 安全性这个概念，实际上， HTTPS 并不是一个新的应用层协议，它其实就是 HTTP + TLS/SSL 协议组合而成，而安全性的保证正是 TLS/SSL 所做的工作。
![](/images/http/2.jpg)

### 区别

* HTTP 是未经安全加密的协议，它的传输过程容易被攻击者监听、数据容易被窃取、发送方和接收方容易被伪造；而 HTTPS 是安全的协议，它通过 **密钥交换算法 - 签名算法 - 对称加密算法 - 摘要算法** 能够解决上面这些问题。
* HTTP 的默认端口是 80，而 HTTPS 的默认端口是 443。

## UDP 和 TCP 的区别

TCP 和 UDP 都位于计算机网络模型中的运输层，它们负责传输应用层产生的数据。

### UDP 定义

UDP 的全称是 <font color=red>User Datagram Protocol</font>，用户数据报协议。它不需要所谓的<font color=red>握手</font>操作，从而加快了通信速度，允许网络上的其他主机在接收方同意通信之前进行数据传输。

### UDP 特点

* UDP 能够支持容忍数据包丢失的带宽密集型应用程序
* UDP 具有低延迟的特点
* UDP 能够发送大量的数据包
* UDP 能够允许 DNS 查找，DNS 是建立在 UDP 之上的应用层协议

### TCP 定义

TCP 的全称是 <font color=red>Transmission Control Protocol</font>，传输控制协议。它能够帮助你确定计算机连接到 Internet 以及它们之间的数据传输。通过三次握手来建立 TCP 连接，三次握手就是用来启动和确认 TCP 连接的过程。一旦连接建立后，就可以发送数据了，当数据传输完成后，会通过关闭虚拟电路来断开连接。

### TCP 特点

* TCP 能够确保连接的建立和数据包的发送
* TCP 支持错误重传机制
* TCP 支持拥塞控制，能够在网络拥堵的情况下延迟发送
* TCP 能够提供错误校验和，甄别有害的数据包

### TCP 和 UDP 的不同


| TCP | UDP |
| - | - | - |
| 面向连接 | 无连接 |
| 会按照特定顺序重新排列数据包 | 数据包没有固定顺序，所有数据包都相互独立 |
| 传输的速度比较慢 | 传输慢 |
|  头部字节有 20 字节 | 头部字节只有 8 字节 |
|  会进行错误校验，并能够进行错误恢复 |  也会错误检查，但会丢弃错误的数据包。 |
|  会使用握手协议，例如 SYN，SYN-ACK，ACK | 无握手协议 |
| 可靠的，因为它可以确保将数据传送到路由器。| 不能保证将数据传送到目标。|

## TCP 三次握手和四次挥手

### TCP 三次握手

![](/images/http/3.jpg)
最开始的时候客户端和服务器都是处于CLOSED状态。主动打开连接的为客户端，被动打开连接的是服务器。

### 为什么需要三次握手

1. 第一次握手：客户端发送网络包，服务端收到了。<font color=red>服务端</font>得出结论：客户端的发送能力、服务端的接收能力。
2. 服务端发包，客户端收到了。<font color=red>客户端</font>得出结论：服务端的接收、发送能力，客户端的接收、发送能力。 从客户端的视角来看，我接到了服务端发送过来的响应数据包，说明服务端接收到了我在第一次握手时发送的网络包，并且成功发送了响应数据包，这就说明，服务端的接收、发送能力正常。而另一方面，我收到了服务端的响应数据包，说明我第一次发送的网络包成功到达服务端，这样，我自己的发送和接收能力也是正常的。
3. 第三次握手：客户端发包，服务端收到了。这样<font color=red>服务端</font>得出结论：客户端的接收、发送能力，服务端的发送、接收能力。 第一、二次握手后，服务端并不知道客户端的接收能力以及自己的发送能力是否正常。而在第三次握手时，服务端收到了客户端对第二次握手作的回应。从服务端的角度，我在第二次握手时的响应数据发送出去了，客户端接收到了。所以，我的发送能力是正常的。而客户端的接收能力也是正常的。


### TCP 四次挥手
![](/images/http/4.jpg)

最开始的时候客户端和服务器都是处于CLOSED状态。主动打开连接的为客户端，被动打开连接的是服务器

### 为什么客户端最后还要等待2MSL？

MSL（Maximum Segment Lifetime）

1. 保证客户端发送的最后一个ACK报文能够到达服务器，因为这个ACK报文可能丢失，站在服务器的角度看来，我已经发送了FIN+ACK报文请求断开了，客户端还没有给我回应，应该是我发送的请求断开报文它没有收到，于是服务器又会重新发送一次，而客户端就能在这个2MSL时间段内收到这个重传的报文，接着给出回应报文，并且会重启2MSL计时器。

2. 防止类似与“三次握手”中提到了的“已经失效的连接请求报文段”出现在本连接中。客户端发送完最后一个确认报文后，在这个2MSL时间中，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失。这样新的连接中不会出现旧连接的请求报文。

### 为什么建立连接是三次握手，关闭连接确是四次挥手呢？

建立连接的时候， 服务器在LISTEN状态下，收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。
而关闭连接时，服务器收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，而自己也未必全部数据都发送给对方了，所以己方可以立即关闭，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，己方ACK和FIN一般都会分开发送，从而导致多了一次。

### 如果已经建立了连接，但是客户端突然出现故障了怎么办？

TCP还设有一个保活计时器，显然，客户端如果出现故障，服务器不能一直等下去，白白浪费资源。服务器每收到一次客户端的请求后都会重新复位这个计时器，时间通常是设置为2小时，若两小时还没有收到客户端的任何数据，服务器就会发送一个探测报文段，以后每隔75秒发送一次。若一连发送10个探测报文仍然没反应，服务器就认为客户端出了故障，接着就关闭连接。


## 简述 HTTP1.0/1.1/2.0 的区别

### HTTP 1.0

* HTTP 1.0 被设计用来使用<font color=red>短链接</font>，即**每次发送数据都会经过 TCP 的三次握手和四次挥手，效率比较低。**
* HTTP 1.0 只使用 header 中的 If-Modified-Since 和 Expires 作为缓存失效的标准。
* HTTP 1.0 不支持断点续传，也就是说，每次都会传送全部的页面和数据。
* HTTP 1.0 认为每台计算机只能绑定一个 IP，所以请求消息中的 URL 并没有传递主机名（hostname）。

### HTTP 1.1

* <font color=red>缓存处理</font>：在HTTP1.0中主要使用header里的If-Modified-Since,Expires来做为缓存判断的标准，HTTP1.1则引入了更多的缓存控制策略例如E-tag，If-Unmodified-Since, If-Match, If-None-Match等更多可供选择的缓存头来控制缓存策略。(强缓存与协商缓存)
* <font color=red>带宽优化及网络连接的使用–断点续传</font>：TTP1.0中，存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能，HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206（Partial Content），这样就方便了开发者自由的选择以便于充分利用带宽和连接。
* <font color=red>长连接</font>，HTTP 1.1支持长连接（PersistentConnection）和请求的流水线（Pipelining）处理，在一个TCP连接上可以传送多个HTTP请求和响应，减少了建立和关闭连接的消耗和延迟，在HTTP1.1中默认开启Connection： **keep-alive**，一定程度上弥补了HTTP1.0每次请求都要创建连接的缺点。
* <font color=red>管线化</font>，**客户端可以同时发出多个HTTP请求，而不用一个个等待响应（有队头阻塞问题）**

### HTTP 2.0 

```xml
帧：HTTP2.0通信的最小单位，所有帧都共享一个8字节的首部，其中包含帧的长度、类型、标志、还有一个保留位，并且至少有标识出当前帧所属的流的标识符，帧承载着特定类型的数据，如HTTP首部、负荷、等等。
消息：比帧大的通讯单位，是指逻辑上的HTTP消息，比如请求、响应等。由一个或多个帧组成
流：比消息大的通讯单位。是TCP连接中的一个虚拟通道，可以承载双向的消息。每个流都有一个唯一的整数标识符
```
* <font color=red>二进制分帧</font>，HTTP 2.0 使用了更加靠近 TCP/IP 的二进制格式，而抛弃了 ASCII 码，提升了解析效率。HTTP2.0通过在应用层和传输层之间增加一个二进制分帧层，突破了HTTP1.1的性能限制、改进传输性能。http2.0将传输的消息分割未更小的帧，并采用二进制的编码。其中首部信息被封装到Headers帧，而我们的request body封装到data帧。
* <font color=red>头部压缩</font>，由于 HTTP 1.1 经常会出现 User-Agent、Cookie、Accept、Server、Range 等字段可能会占用几百甚至几千字节，而 Body 却经常只有几十字节，所以导致头部偏重。http2.0在客户端和服务器共同维护**一个头部表head list**，由客户端和服务端一起维护。对于相同没用变化的头部，不再传输，每次只将有变化的头部封装到headers frame中。新增和修改的头部会被追加到头部表中。HTTP2.0使用**encoder**来减少需要传输的header大小。
* <font color=red>强化安全</font>，由于安全已经成为重中之重，所以 HTTP2.0 一般都跑在 HTTPS 上。
* <font color=red>多路复用</font>，http2.0将消息分解为独立帧，交错发送，然后在另外一端重新组装，这样一个连接上就可以存在多个请求和响应。
* <font color=red>服务器推送</font>，http2.0将消息分解为独立帧，交错发送，然后在另外一端重新组装，这样一个连接上就可以存在多个请求和响应。

## dns 解析

1. <font color=red>递归查询</font> 返回的结果只有两种:查询成功或查询失败, 主机向本地域名服务器的查询一般都是采用 
2. <font color=red>迭代查询</font> 又称作重指引,返回的是最佳的查询点或者主机地址, 本地域名服务器向根域名服务器的查询


## HTTP的请求方法

* GET: 获取URL指定的资源；
* POST：传输实体信息
* PUT：上传文件
* DELETE：删除文件
* HEAD：获取报文首部，与GET相比，不返回报文主体部分
* OPTIONS：询问支持的方法（跨域 预检）
* TRACE：追踪请求的路径；
* CONNECT：要求在与代理服务器通信时建立隧道，使用隧道进行TCP通信。主要使用SSL和TLS将数据加密后通过网络隧道进行传输。

## XSS  跨站脚本攻击 && CSRF 跨站请求伪造

### XSS(cross-site scripting) 跨站脚本攻击
类型：
1. <font color=red>存储型 XSS</font> （用户保存数据的网站功能，如论坛发帖、商品评论、用户私信等）
1. <font color=red>反射型 XSS </font>  `a.com?content=<script>alert(“xss”)</script>`
3. <font color=red>DOM 型 XSS</font> 取出和执行恶意代码由浏览器端完成，属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞

防范：
1. httponly 
2. 输入的过滤 
3. 拼接html时，转义html如replace(/&/g, '&amp;') 
4. 避免使用 innerHTML(v-html)等 

###  CSRF(cross-site requst forgery) 跨站请求伪造
诱导用户打开网站，该网站嵌入代码，以下两个都不受同源策略。
1. get 请求 `<img src="xxx.com/delete/:id/:arcitle" />`
2. 创建自动提交form 表单。

防范：
1. 验证码 
2. Reffer校验 
3. 随机Token校验 


## HTTP 常见状态码

* <font color=red>200成功</font> 服务器已成功处理了请求。
* <font color=red>301永久重定向</font> 请求的网页已永久移动到新位置。服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置。
  * 你将永久更改网页的 URL时。
  * 你将永久迁移到新域名时。
  * 当你从 HTTP 切换到 HTTPS 时。
  * 你希望修复非 www / www 重复内容问题时。
  * 永久合并两个或多个页面或网站时。
  * 你将永久更改网站的 URL 结构时。
* <font color=red>308永久重定向</font> 308与301定义一致，唯一的区别在于，308状态码不允许浏览器将原本为POST的请求重顶到GET请求上。

* <font color=red>302临时重定向</font> 允许各种各样的重定向，一般都实现为GET到GET重定向，但是不能确保POST会重定向为POST。
  * 当你想将用户重定到正确的网站版本（基于位置/语言）时。
  * 当你要对网页的功能或设计进行 A / B 拆分测试时。
  * 你希望在不影响旧页面排名的情况下获得新页面的反馈时。
  * 当你正在进行促销，并希望暂时将访问者重定向到促销页面时。
* <font color=red>303临时重定向</font> 只允许任意请求到GET的重定向。
* <font color=red>307临时重定向</font> 307和302一样，但不允许POST到GET的重定向。
* <font color=red>304协商缓存</font> 
* <font color=red>400错误请求</font> 
* <font color=red>401未认证响应</font>   是由于用户没有进行身份认证或者身份认证不对。
* <font color=red>403拒绝响应</font> 是当用户通过了身份验证，但无权对给定资源执行请求的操作（比如没有读写权限）。
* <font color=red>404未找到资源</font>  
* <font color=red>500服务器内部错误</font> 服务器端在执行请求时发生了错误


## HTTPS工作原理

### 什么是 HTTPS

<font color=red>HTTPS</font> 是在<font color=red>HTTP</font>上建立<font color=red>SSL</font>加密层，并对传输数据进行加密，是HTTP协议的安全版。 说得再简单一点，<font color=red>HTTPS</font>就是 <font color=red>HTTP + SSL</font>:
![](/images/http/6.png)


<font color=red>SSL</font>又是是什么？

![](/images/http/5.png)

<font color=red>HTTP</font>是应用层的协议，<font color=red>HTTP</font>是直接和<font color=red>TCP</font>通信的，当我们更换为<font color=red>HTTPS</font>的时候，它就演变成了，先和SSL</font>通信、然后再由<font color=red>SSL</font>和<font color=red>TCP</font>通信了。知道了这一流程，我们思考一下，加密过程是在哪个过程进行的？
结果很显然，加密过程就是在<font color=red>HTTP-><font color=red>SSL</font>这一阶段实现的，所以SSL简单来说，就是实现<font color=red>HTTP</font>的加密过程:

![](/images/http/7.png)


### 对称加密

简单说就是有一个<font color=red>密钥</font>，它可以<font color=red>加密</font>数据，也可以对加密后的数据进行<font color=red>解密</font>。

<font color=red>对称加密</font>就是通信双方都有一把同样的钥匙，用于打开同一个<font color=red>解密</font>，从而获取数据的一种方式:

![](/images/http/8.png)


### 非对称加密

简单的来说就是有两把密钥，一把叫做<font color=red>公钥</font>、一把叫<font color=red>私钥</font>。用公钥加密的内容必须用私钥才能解开，同理，私钥加密的内容只有公钥能解开。公钥对于私钥来说是非对称的，所以我们把这种加密方式称为非对称加密：

![](/images/http/9.png)


### HTTPS用对称加密可行吗

不可行。
如果通信双方（浏览器和服务器）都各自拥有同样的私钥，在发送方发送数据之前，将数据<font color=red>加密</font>起来，然后接收方再用私钥<font color=red>解密</font>，这样就能完美的保证通信的安全。但是关键的问题就在于，在通信之前，服务器或者浏览器如何同时拥有一把同样的私钥呢？
服务器给浏览器传输私钥。 当浏览器发起请求到服务器时，服务器生成一个私钥，然后传输给浏览器，浏览器拿到之后，他们正式通信就开始用这一个私钥进行加密通信。**若被中间人挟持，拿到私钥之后，就可解密通信，即不安全**。

### HTTPS用非对称加密可行吗

我们利用非对称加密来模拟一下浏览器和服务器通信的过程.
浏览器向服务器发起请求，服务器收到请求之后，将公钥传输给浏览器，然后在接下来的通信中，浏览器要向服务器发送数据时，先用公钥将数据加密，然后发送给服务器，服务器收到之后，再用对应的私钥进行解密，这样就保证了<font color=red>浏览器->服务器</font>这条路的数据安全。因为浏览器发送给服务器的数据，只有服务器有私钥能够解密它。那么反过来，<font color=red>服务器->浏览器</font>这条路的通信是否安全呢？**若被中间人挟持，拿到公钥之后，就可解密 服务器->浏览器 通信，即不安全**。


若浏览器预存了一套 <font color=red>公钥</font> 和 <font color=red>私钥</font>，服务器也预存了一套 <font color=red>公钥</font> 和 <font color=red>私钥</font>，都先将自己的公钥发送给对方，虽然能保证安全。但非对称加密其实是非常耗时的，当浏览器向服务器发起请求时，用户肯定是希望越早看到响应数据越好，所以也不可取。


### 非对称加密（服务器 公钥+私钥） + 对称加密（浏览器生成密钥）

1、浏览器向服务器发起请求；
2、服务器将自己的 <font color=red>公钥A+</font> 返回给浏览器；
3、浏览器在本地生成一个<font color=red>密钥B</font>，然后利用 <font color=red>公钥A+</font> 对<font color=red>密钥B</font>进行加密，加密之后传输给服务器；
4、服务器收到数据，利用 <font color=red>私钥A-</font>对数据进行解密，拿到浏览器生成的<font color=red>密钥B</font>；
5、之后双方的通信，都用<font color=red>密钥B</font>进行。

若是被中间人挟持： 

![](/images/http/10.png)

>  注意：此时中间人已经拿到了服务器的公钥A+和浏览器的密钥X


### 数字证书

<font color=red>CA机构</font> 就是它会给网站颁发<font color=red>身份证</font>。当某个网站想要启用HTTPS协议的时候，需要向CA机构申请一份证书，这个证书就是数字证书。
<font color=red>数字证书包括了证书持有者信息、公钥和有效期等等信息。</font>
有了这个证书之后，当浏览器向目标网站服务器发起请求时，服务器只需要把数字证书返回给浏览器就行了。因为是目标网站有数字证书，如同身份证一样，那浏览器就可以认为，返回数据给我的，就是这个网站。
但证书本身传输的过程中，万一被中间人挟持并修改，会造成数据不安全。



### 数字签名

假设CA机构（不是服务器）有一套非对称加密的 <font color=red>公钥A+</font> 和 <font color=red>私钥A-</font>
1. 网站向CA机构申请颁发数字证书；
2. CA机构通过审核之后，会生成一份证书数据（此时为明文，内容包括：证书持有者信息、网站公钥、和有效期等）；
3. 利用散列函数对证书的明文数据进行<font>Hash</font>处理，生成一份<font color=red>数据摘要</font>；
4. 接着利用<font color=red>私钥A-</font>对这份<font color=red>数据摘</font>要进行加密，得到<font color=red>数字签名</font>；
5. 将<font color=red>证书明文数据</font> + <font color=red>数字签名</font> 合并到一起，组成完整的<font color=red>数字证书</font>;
5. 将这份数字证书颁发给对应的网站。

![](/images/http/11.png)

其实<font color=red>数字签名</font>就是CA机构利用自有的一套**非对称的私钥对证书数据进行加密之后的数据**。

那么现在我们有了数字签名加持的数字证书，浏览器在接收到的时候，如何进行验证呢？

> 需要注意的是，我们的浏览器已经预存了CA机构的<font color=red>公钥</font>，这把公钥的作用就是用来解锁<font color=red>数字签名</font>的。

1、浏览器向目前网站服务器发起请求；
2、目标服务器把<font color=red>数字证书</font>返回给浏览器（包括证书明文数据 + 数字签名）；
3、浏览器拿到数字证书之后，先拿到<font color=red>签名</font>，利用浏览器预存的CA机构的公钥，对<font color=red><font color=red>签名</font>进行解密，解密之后得到一份<font color=red>数据摘要</font>（假设叫<font color=red>T</font>），接着利用证书里面提供的Hash算法对明文数据进行<font color=red>Hash</font>，又得到一份<font color=red>数据摘要</font>（假设叫<font color=red>S</font>）。此时，如果 <font color=red>T=S</font> ，那浏览器认为这份证书就是有效的，否则无效。

![](/images/http/12.png)

## HTTPS完整的工作流程


假设现在CA机构有一套非对称加密的<font color=red>公钥A+S</font> 和 <font color=red>私钥A-</font>（浏览器会预存<font color=red>公钥A+</font>）
目标服务器也有一套非对称加密的<font color=red>公钥B+</font> 和 <font color=red>私钥B-</font>
1、浏览器向服务器发起请求；
2、目标服务器收到请求，将<font color=red>数字证书</font>返回给浏览器（包括<font color=red>公钥B+</font> + 数字签名）；
3、浏览器收到证书之后，先取到<font color=red>签名</font>，利用浏览器预存的CA机构的<font color=red>公钥A+</font>，对<font color=red>签名</font>进行解密，解密之后得到一份<font color=red>数据摘要</font>（假设叫<font color=red>T</font>），接着利用证书里面提供的<font color=red>Hash算法</font>对<font color=red>明文数据</font>进行<font color=red>Hash</font>，又得到一份数据摘要（假设叫<font color=red>S</font>）。此时，如果 <font color=red>T=S</font> ，那认为这份证书就是有效的。有效则进行下一步，否则直接断开连接；
4、浏览器从证书里面取出目标网站的<font color=red>公钥B+</font>，然后在本地生成一个<font color=red>密钥X</font>，接着利用<font color=red>公钥B+</font> 对<font color=red>密钥X</font>进行加密，加密之后传输给服务器；
5、服务器收到数据，利用<font color=red>私钥B-</font>对数据进行解密，拿到浏览器生成的<font color=red>密钥X</font>；
6、之后双方的通信，都用<font color=red>密钥X</font>加密之后进行。




## 参考链接

* [http2 http1.1 网络加载对比](https://http2.akamai.com/demo)
* [HTTP中的301、302、303、307、308](https://www.cnblogs.com/amyzhu/p/11763438.html)
* [DNS递归查询与迭代查询](https://www.cnblogs.com/qingdaofu/p/7399670.html)
* [XSS 攻击](https://tech.meituan.com/2018/09/27/fe-security.html)
* [知乎  HTTP/2 无解决TCP队首阻塞问题，仅仅是通过多路复用解决了以前 HTTP1.1 管线化请求时的队首阻塞](https://www.zhihu.com/question/65900752/answer/255085226)
* [http2.0 讲解](https://juejin.cn/post/6844903545532071943)
* [http2.0 讲解](https://juejin.cn/post/6844903545532071943)
* [http1.1 && http2.0 总结](https://wangpengcheng.github.io/2020/03/12/http_interview/)
* [知乎 三次握手，四次挥手](https://zhuanlan.zhihu.com/p/53374516)
* [csdn 三次握手、四次挥手](https://blog.csdn.net/qzcsu/article/details/72861891)
* [DNC、TCP详情常见问题](https://juejin.cn/post/6956046759428636708)

