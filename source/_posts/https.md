---
title: https
date: 2021-04-25 15:37:34
categories: 
- web前端
tags:
- http https
---

## 什么是 HTTPS

<font color=red>HTTPS</font> 是在<font color=red>HTTP</font>上建立<font color=red>SSL</font>加密层，并对传输数据进行加密，是HTTP协议的安全版。 说得再简单一点，<font color=red>HTTPS</font>就是 <font color=red>HTTP + SSL</font>:
![](/images/http/6.png)


<font color=red>SSL</font>又是是什么？

![](/images/http/5.png)

<font color=red>HTTP</font>是应用层的协议，<font color=red>HTTP</font>是直接和<font color=red>TCP</font>通信的，当我们更换为<font color=red>HTTPS</font>的时候，它就演变成了，先和SSL</font>通信、然后再由<font color=red>SSL</font>和<font color=red>TCP</font>通信了。知道了这一流程，我们思考一下，加密过程是在哪个过程进行的？
结果很显然，加密过程就是在<font color=red>HTTP-><font color=red>SSL</font>这一阶段实现的，所以SSL简单来说，就是实现<font color=red>HTTP</font>的加密过程:

![](/images/http/7.png)


## 对称加密

简单说就是有一个<font color=red>密钥</font>，它可以<font color=red>加密</font>数据，也可以对加密后的数据进行<font color=red>解密</font>。

<font color=red>对称加密</font>就是通信双方都有一把同样的钥匙，用于打开同一个<font color=red>解密</font>，从而获取数据的一种方式:

![](/images/http/8.png)


## 非对称加密

简单的来说就是有两把密钥，一把叫做<font color=red>公钥</font>、一把叫<font color=red>私钥</font>。用公钥加密的内容必须用私钥才能解开，同理，私钥加密的内容只有公钥能解开。公钥对于私钥来说是非对称的，所以我们把这种加密方式称为非对称加密：

![](/images/http/9.png)


## HTTPS用对称加密可行吗

不可行。
如果通信双方（浏览器和服务器）都各自拥有同样的私钥，在发送方发送数据之前，将数据<font color=red>加密</font>起来，然后接收方再用私钥<font color=red>解密</font>，这样就能完美的保证通信的安全。但是关键的问题就在于，在通信之前，服务器或者浏览器如何同时拥有一把同样的私钥呢？
服务器给浏览器传输私钥。 当浏览器发起请求到服务器时，服务器生成一个私钥，然后传输给浏览器，浏览器拿到之后，他们正式通信就开始用这一个私钥进行加密通信。**若被中间人挟持，拿到私钥之后，就可解密通信，即不安全**。

## HTTPS用非对称加密可行吗

我们利用非对称加密来模拟一下浏览器和服务器通信的过程.
浏览器向服务器发起请求，服务器收到请求之后，将公钥传输给浏览器，然后在接下来的通信中，浏览器要向服务器发送数据时，先用公钥将数据加密，然后发送给服务器，服务器收到之后，再用对应的私钥进行解密，这样就保证了<font color=red>浏览器->服务器</font>这条路的数据安全。因为浏览器发送给服务器的数据，只有服务器有私钥能够解密它。那么反过来，<font color=red>服务器->浏览器</font>这条路的通信是否安全呢？**若被中间人挟持，拿到公钥之后，就可解密 服务器->浏览器 通信，即不安全**。


若浏览器预存了一套 <font color=red>公钥</font> 和 <font color=red>私钥</font>，服务器也预存了一套 <font color=red>公钥</font> 和 <font color=red>私钥</font>，都先将自己的公钥发送给对方，虽然能保证安全。但非对称加密其实是非常耗时的，当浏览器向服务器发起请求时，用户肯定是希望越早看到响应数据越好，所以也不可取。


## 非对称加密（服务器 公钥+私钥） + 对称加密（浏览器生成密钥）

1、浏览器向服务器发起请求；
2、服务器将自己的 <font color=red>公钥A+</font> 返回给浏览器；
3、浏览器在本地生成一个<font color=red>密钥B</font>，然后利用 <font color=red>公钥A+</font> 对<font color=red>密钥B</font>进行加密，加密之后传输给服务器；
4、服务器收到数据，利用 <font color=red>私钥A-</font>对数据进行解密，拿到浏览器生成的<font color=red>密钥B</font>；
5、之后双方的通信，都用<font color=red>密钥B</font>进行。

若是被中间人挟持： 

![](/images/http/10.png)

>  注意：此时中间人已经拿到了服务器的公钥A+和浏览器的密钥X


## 数字证书

<font color=red>CA机构</font> 就是它会给网站颁发<font color=red>身份证</font>。当某个网站想要启用HTTPS协议的时候，需要向CA机构申请一份证书，这个证书就是数字证书。
<font color=red>数字证书包括了证书持有者信息、公钥和有效期等等信息。</font>
有了这个证书之后，当浏览器向目标网站服务器发起请求时，服务器只需要把数字证书返回给浏览器就行了。因为是目标网站有数字证书，如同身份证一样，那浏览器就可以认为，返回数据给我的，就是这个网站。
但证书本身传输的过程中，万一被中间人挟持并修改，会造成数据不安全。



## 数字签名

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

* [https 如何通俗易懂的给你讲明白HTTPS？](https://juejin.cn/post/6955767063524671524)
* [彻底搞懂HTTPS的加密原理](https://zhuanlan.zhihu.com/p/43789231)

