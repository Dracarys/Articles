# 面试题系列之网络协议

## 1. 网络协议基础

### 1.1 七层模型
1. 物理层： 双绞线、无线 2.4G、5G、光线
2. 数据链路层：以太网
3. 网络层：ARP、IPv4、IPv6、ICMP、IPsec
4. 传输层：TCP、UDP、UDP-Lite、SCTP、DCCP
5. 会话层：
6. 表示层：
7. 应用层：HTTP、SSH、FTP、DNS、TELNET、SSL/TLS、SIP

### 1.2 ping 是什么协议
ICMP（Internet Control and Message Protocal）

### 1.3 传输层和网络层分别是做什么的？
- 网络层，用来解决从哪来，到哪去的问题，即在浩瀚网海之中如何准确传递消息
- 传输层，用来传输数据，在网络层的基础之上进行交流，解决消息如何传递的问题。
- 应用层，解决消息如何封装和读取的问题

## 2. IP 协议

## 3. 传输层协议
TCP/UDP 的区别，头部对比：

| TCP | UDP |
|:----|:----|
|面向链接的|无连接|
|可靠的协议|不可靠的|
|效率相对较低|高速、实时性好|
|顺序控制|无|
|重发控制|无|
|流量控制|无|
|拥塞控制|无|
|20个字节，加上可选就是24个字节|固定 8个字节|
### 3.1 TCP 协议

#### 3.1.1 TCP的三次握手四次挥手

MSS Maximum Segments Size 最大消息段长度就是在三次握手的过程中确立的。
建立连接的三次握手过程：

![TCP建立连接](https://ask.qcloudimg.com/http-save/yehe-1446775/uwl5mh9ogf.png?imageView2/2/w/1620)

1. 客户端发送的 TCP 报文中标识位 SYN 置1，初始序号 seq=x（随机）。Client 进入SYN_SENT状态，等待 Server 确认；
2. 服务器收到数据包后，根据标识位 SYN=1 知道 Client 请求建立连接，Server将标识位 SYN 和 ACK 都置为1， ack=x+1，随机产生一个初始序列号 seq=y，并将该数据不傲发送给 Client 以确认建立连接请求，Server 进入 SYN_RCVD 状态；
3. Client收到确认后，检查ack是否为x+1，ACK是否为1，如果正确则将标志位ACK置为1，ack=y+1，并将该数据包发送给Server。Server检查ack是否为y+1，ACK是否为1，如果正确则连接建立成功，Client和Server进入ESTABLISHED状态，完成三次握手，随后Client与Server之间可以开始传输数据了

断开连接的四次回收过程：

![TCP断开连接](https://ask.qcloudimg.com/http-save/yehe-1446775/ok7sodrcza.png?imageView2/2/w/1620)

1. Client发送一个FIN，用来关闭Client到Server的数据传送，Client进入FIN_WAIT_1状态
2. Server收到FIN后，发送一个ACK给Client，确认序号为u + 1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态
3. Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST_ACK状态
4. Client收到FIN后，Client进入TIME_WAIT状态(主动关闭方才会进入该状态），接着发送一个ACK给Server，确认序号为w + 1，Server进入CLOSED状态，完成四次挥手


#### 3.1.2 TCP滑动窗口，窗口大小，如何确定的？
TCP 协议为了达到更高效的数据发送效率, 会使用一个叫滑动窗口的机制来批量发送数据, 其形式如图：
![TCP滑动窗口](http://user-image.logdown.io/user/12785/blog/12037/post/730737/TsL7H0lRS9W8qRXq1xdS_滑动窗口1)

窗口大小是如何确定的呢？
tcp 有个机制叫慢启动. 一般来讲, 如果 MSS 为 1460 Bytes (以太网标准), 则窗口初始大小设置为 3 MSS 大小. 此后, 根据每次数据往返的 ack, 会逐渐增大这个值. 首次开始数据传输时, 拥塞窗口大小会以 1, 2, 4 这样的指数倍增长, 直到发生超时重发, 窗口会重新开始计算, 并设置一个慢启动阈值, 这个阈值为窗口大小的一半, 当窗口大小超过阈值时, 窗口的增长速率按照一个小于 MSS 大小的比例增长。

如果遇超时重传的情况, 则拥塞窗口大小会重新设置为 (慢启动阈值 + 3 MSS). 而阈值会重新设置为当前窗口的一般. 窗口变化如图：

![TCP拥塞控制](http://user-image.logdown.io/user/12785/blog/12037/post/730737/sqeDKQS8SKUrvlqIxrVn_拥塞窗口)

那么, 最终的窗口大小其实是由接收方的窗口控制参数和拥塞窗口大小共同决定的, 实际发送的窗口数据量大小, 是两者中的较小值.

#### 3.1.4 TCP是如何保证可靠的

TCP 的每个分断都有自己的序号，接受端收到一个分段后会向发送方确认，并附带下一分段的编号，发送收到确认后，就会知道该编号以前到接受方都已经收到了。如果超时未收到确认，那么发送方将重发。

### 3.2 UDP 协议

### 3.2.3 UDP可以实现一对多吗？怎么实现？

## 4. HTTP 协议
HTTP协议（HyperTextTransferProtocol，超文本传输协议）是用于从WWW服务器传输超文本到本地浏览器的传输协议

[引用](https://cloud.tencent.com/developer/article/1351472)

**无状态协议**

**怎么解决无状态**
### 4.1 URI 和 URL

- URI(Uniform Resource Identifier)，统一资源标识符，用来唯一地标识一个资源；
- URL（Uniform Resource Locator），统一资源定位器，它是一种具体的 URI，即 URL 可以用来标识一个资源，而且还指明了如何定位这个资源。

#### 4.1.1 URI
组成部分：

1. 访问资源的命名机制
2. 存放资源的主机名
3. 资源自身的名称，由路径表示，着重强调于资源

#### 4.1.2 URL
组成部分：

1. 协议(或称为服务方式)
2. 存有该资源的主机IP地址(有时也包括端口号)
3. 主机资源的具体地址。如目录和文件名等

### 4.1 HTTP 报文格式

#### 4.1.1 请求报文
![请求报文格式](https://img-blog.csdn.net/20170330192653242?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSG9sbW9meQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 4.1.2 响应报文
![响应报文格式](https://img-blog.csdn.net/20170330192754102?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSG9sbW9meQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 4.1.3 常用头部

**通用首部字段（请求报文与响应报文都会使用的首部字段）**

- Date：创建报文时间
- Connection：连接的管理
- Cache-Control：缓存的控制
- Transfer-Encoding：报文主体的传输编码方式

**请求首部字段（请求报文会使用的首部字段）**

- Host：请求资源所在服务器
- Accept：可处理的媒体类型
- Accept-Charset：可接收的字符集
- Accept-Encoding：可接受的内容编码
- Accept-Language：可接受的自然语言

**响应首部字段（响应报文会使用的首部字段）**

- Accept-Ranges：可接受的字节范围
- Location：令客户端重新定向到的URI
- Server：HTTP服务器的安装信息

**实体首部字段（请求报文与响应报文的的实体部分使用的首部字段）**

- Allow：资源可支持的HTTP方法
- Content-Type：实体主类的类型
- Content-Encoding：实体主体适用的编码方式
- Content-Language：实体主体的自然语言
- Content-Length：实体主体的的字节数
- Content-Range：实体主体的位置范围，一般用于发出部分请求时使用

### 4.2 HTTP 缓存控制
缓存控制头部：Cache-Control，其值详如下：

- max-stale 缓存可以随意提供过期的文件
- max-stale=\<s\> 增加时间秒，改时间内不能过期
- min-refresh=\<s\> 至少在未来指定的秒数内要保持新鲜。
- max-age=\<s\> 无法返回缓存时间长于 s 秒的文档
- no-cache 除非在验证，否则不接受缓存
- no-store 缓存应该尽快从过年存储中删除文档的所有痕迹
- only-if-cached 只有当缓存中有副本时，才会获取一份副本。

缓存新鲜度验证条件头部：

- If-Modified-Since:<date> 如果从指定日期之后文档被修改过，就执行该方法。可以与 Last-Modified 服务器响应头部配合使用。
- If-None-Match:<tags> 如果与标签不同，就执行请求方法。

弱验证器，即允许不更新缓存的变化度。

### 4.2 HTTP请求有哪些方法？如何选择？

- GET 用来向服务器发送请求
- POST 用于将数据发送给服务以创建和更新资源
- HEAD 与 GET 类似，但是没有响应题，仅传输状态行和标题部分
- OPTION 用来描述了目标资源的通信选项
- PUT 向服务器插入内容
- DELETE 向服务器删除内容
- CONNECT 用来建立到给定URI标识的服务器的隧道；它通过简单的TCP / IP隧道更改请求连接，通常实使用解码的HTTP代理来进行SSL编码的通信
- TRACE 用于沿着目标资源的路径执行消息环回测试 

### 4.3 cookie
Cookie是什么？ Cookie 是一小段文本信息，伴随着用户请求和页面在 Web 服务器和浏览器之间传递。Cookie 包含每次用户访问站点时 Web 应用程序都可以读取的信息。

为什么需要Cookie？ 因为HTTP协议是无状态的，对于一个浏览器发出的多次请求，WEB服务器无法区分 是不是来源于同一个浏览器。所以，需要额外的数据用于维护会话。 Cookie 正是这样的一段随HTTP请求一起被传递的额外数据。

Cookie能做什么？ Cookie只是一段文本，所以它只能保存字符串。而且浏览器对它有大小限制以及 它会随着每次请求被发送到服务器，所以应该保证它不要太大。 Cookie的内容也是明文保存的，有些浏览器提供界面修改，所以， 不适合保存重要的或者涉及隐私的内容。

Cookie 的限制。 大多数浏览器支持最大为 4096 字节的 Cookie。由于这限制了 Cookie 的大小，最好用 Cookie 来存储少量数据，或者存储用户 ID 之类的标识符。用户 ID 随后便可用于标识用户，以及从数据库或其他数据源中读取用户信息。 浏览器还限制站点可以在用户计算机上存储的 Cookie 的数量。大多数浏览器只允许每个站点存储 20 个 Cookie；如果试图存储更多 Cookie，则最旧的 Cookie 便会被丢弃。有些浏览器还会对它们将接受的来自所有站点的 Cookie 总数作出绝对限制，通常为 300 个。

通过前面的内容，我们了解到Cookie是用于维持服务端会话状态的，通常由服务端写入，在后续请求中，供服务端读取。 下面本文将按这个过程看看Cookie是如何从服务端写入，最后如何传到服务端以及如何读取的。

Cookie 可以笼统的分为两类：

- 会话 cookie 一种临时 cookie，记录用户访问站点是的设置和偏好。推出浏览时要删除
- 持久 cookie 生存时间更长，存储在硬盘上，即使重启也仍然保留。用于维护用户会周期性访问的站点的配置文件或登录名。

通过下面两个响应头在客户端设置 cookie

- Set-Cookie
- Set-Cookie2

#### cookie 的域属性
Cookie 通常是谁创建，才会发送给谁。产生 Cookie 的服务器可以向 Set-Cookie 响应头添加一个 Domain 属性来控制哪些站点可以看到哪个 cookie。例如：

```javascript
Set-cookie: user="mary17"; domain="airtravelbargains.com"
```

#### cookie 路径属性
目的与域类似，只是更细化：

```javascript
Set-cookie: user="mary17"; domain="airtravelbargains.com"; path=/auto
```


### 网关中的 HTTP 隧道

HTTP/1.1 规范保留了 CONNECT 方法，但没有对其功能进行描述

CONNECT 方法请求隧道网管创建一条到达任意目的服务器和端口的 TCP 连接（隧道），并对客户端和服务器之间的后续数据进行忙转发。

一旦 TCP 连接建立，网关会发送一条 HTTP 200 Connction Established 响应来通知客户端。注意该响应与普通的响应不同，不需要包含 Content-Type 头部。因此此连接针对的是字节流，不是涉及内容。

至此，隧道建立完毕。

在隧道建立之前，可以要求客户端提供认证信息，通过后再予以建立。

### 中继
HTTP 中继（relay）是没有完全遵循 HTTP 规范的简单 HTTP 代理。中继负责处理 HTTP 中建立连接的部分，然后对字节进行盲转发。

### 客户端识别

头部
|头部名称|头部类型|描述|
|:------|:------|:--|
|From|请求|用户的 E-mail 地址|
|User-Agent|请求|用户的浏览器软件|
|Referer|请求|用户是从这个页面上依照连接跳转过来的|
|Authorization|请求|用户名和密码|
|Client-IP|请求|扩展（请求）|客户端的 IP 地址|
|X-Forwarded-For|扩展（请求）|客户端的 IP 地址|
|Cookie|扩展（请求）|服务器产生的 ID 标签|

### HTTP 认证机制

#### 基本认证
1. 客户端正常发送请求；
2. 服务端进行质询，头部带上 WWW-Authenticate，服务器会头部的值中知名需哟啊鉴权的区域，并返回 401 状态码 Unauthorized
3. 客户端收到响应后，进行授权，头部带上 Authorization：服务指定的密钥
4. 服务端响应 200 OK，并在头部带上 Authentication-Info

#### 安全域
安全域（security realm），用于指定可访问的范围。

#### 摘要认证
基本认证存在明文传输的问题，摘要认证正式要解决该问题。摘要认证的安全程度只是略高于基本认证，更安全的是HTTPS

通常是 摘要+盐


## 5 HTTPS 协议
[HTTPS加密协议详解](https://www.wosign.com/faq/faq2016-0309-01.htm)

### 5.1 HTTPS 原理
HTTPS 在真正开始 HTTP 请求之前，先与服务器进行几次握手验证，以确定双方身份，如图：

![HTTPS SSL 建立流程](https://www.wosign.com/News/images/20160309041.png)

验证流程：

(1).client_hello

客户端发起请求，以明文传输请求信息，包含版本信息，加密套件候选列表，压缩算法候选列表，随机数，扩展字段等信息，相关信息如下：

支持的最高TSL协议版本version，从低到高依次 SSLv2 SSLv3 TLSv1 TLSv1.1 TLSv1.2，当前基本不再使用低于 TLSv1 的版本;

客户端支持的加密套件 cipher suites 列表， 每个加密套件对应前面 TLS 原理中的四个功能的组合：认证算法 Au (身份验证)、密钥交换算法 KeyExchange(密钥协商)、对称加密算法 Enc (信息加密)和信息摘要 Mac(完整性校验);

支持的压缩算法 compression methods 列表，用于后续的信息压缩传输;

随机数 random_C，用于后续的密钥的生成;

扩展字段 extensions，支持协议与算法的相关参数以及其它辅助信息等，常见的 SNI 就属于扩展字段，后续单独讨论该字段作用。

(2).server_hello+server_certificate+sever_hello_done

(a) server_hello, 服务端返回协商的信息结果，包括选择使用的协议版本 version，选择的加密套件 cipher suite，选择的压缩算法 compression method、随机数 random_S 等，其中随机数用于后续的密钥协商;

(b)server_certificates, 服务器端配置对应的证书链，用于身份验证与密钥交换;

(c) server_hello_done，通知客户端 server_hello 信息发送结束;

(3).证书校验

客户端验证证书的合法性，如果验证通过才会进行后续通信，否则根据错误情况不同做出提示和操作，合法性验证包括如下：

证书链的可信性 trusted certificate path，方法如前文所述;

证书是否吊销 revocation，有两类方式离线 CRL 与在线 OCSP，不同的客户端行为会不同;

有效期 expiry date，证书是否在有效时间范围;

域名 domain，核查证书域名是否与当前的访问域名匹配，匹配规则后续分析;

(4).client_key_exchange+change_cipher_spec+encrypted_handshake_message

(a) client_key_exchange，合法性验证通过之后，客户端计算产生随机数字 Pre-master，并用证书公钥加密，发送给服务器;

(b) 此时客户端已经获取全部的计算协商密钥需要的信息：两个明文随机数 random_C 和 random_S 与自己计算产生的 Pre-master，计算得到协商密钥;

enc_key=Fuc(random_C, random_S, Pre-Master)

(c) change_cipher_spec，客户端通知服务器后续的通信都采用协商的通信密钥和加密算法进行加密通信;

(d) encrypted_handshake_message，结合之前所有通信参数的 hash 值与其它相关信息生成一段数据，采用协商密钥 session secret 与算法进行加密，然后发送给服务器用于数据与握手验证;

(5).change_cipher_spec+encrypted_handshake_message

(a) 服务器用私钥解密加密的 Pre-master 数据，基于之前交换的两个明文随机数 random_C 和 random_S，计算得到协商密钥:enc_key=Fuc(random_C, random_S, Pre-Master);

(b) 计算之前所有接收信息的 hash 值，然后解密客户端发送的 encrypted_handshake_message，验证数据和密钥正确性;

(c) change_cipher_spec, 验证通过之后，服务器同样发送 change_cipher_spec 以告知客户端后续的通信都采用协商的密钥与算法进行加密通信;

(d) encrypted_handshake_message, 服务器也结合所有当前的通信参数信息生成一段数据并采用协商密钥 session secret 与算法加密并发送到客户端;

(6).握手结束

客户端计算所有接收信息的 hash 值，并采用协商密钥解密 encrypted_handshake_message，验证服务器发送的数据和密钥，验证通过则握手完成;

(7).加密通信

开始使用协商密钥与算法进行加密通信。

注意：

(a) 服务器也可以要求验证客户端，即双向认证，可以在过程2要发送 client_certificate_request 信息，客户端在过程4中先发送 client_certificate与certificate_verify_message 信息，证书的验证方式基本相同，certificate_verify_message 是采用client的私钥加密的一段基于已经协商的通信信息得到数据，服务器可以采用对应的公钥解密并验证;

(b) 根据使用的密钥交换算法的不同，如 ECC 等，协商细节略有不同，总体相似;

(c) sever key exchange 的作用是 server certificate 没有携带足够的信息时，发送给客户端以计算 pre-master，如基于 DH 的证书，公钥不被证书中包含，需要单独发送;

(d) change cipher spec 实际可用于通知对端改版当前使用的加密通信方式，当前没有深入解析;

(e) alter message 用于指明在握手或通信过程中的状态改变或错误信息，一般告警信息触发条件是连接关闭，收到不合法的信息，信息解密失败，用户取消操作等，收到告警信息之后，通信会被断开或者由接收方决定是否断开连接

### 5.2 HTTPS 用到的主要加密算法
非对称加密算法：RSA，DSA/DSS     在客户端与服务端相互验证的过程中用的是对称加密 
对称加密算法：AES，RC4，3DES     客户端与服务端相互验证通过后，以随机数作为密钥时，就是对称加密
HASH算法：MD5，SHA1，SHA256  在确认握手消息没有被篡改时 

### 5.3 HTTPS 密钥协商交还的过程，中间人攻击，即charles抓包的原理
熟悉了上面双方互下确认的过程，在中间截获，并用自己的证书进行替换转发即可。

## 6. 项目经验
### 6.1 网络造成卡顿的原因

拥塞，TCP拥塞控制

### 6.2 断点续传怎么实现？需要设置什么？
断点续传其实正如字面意思，就是在下载的断开点继续开始传输，不用再从头开始。所以理解断点续传的核心后，发现其实和很简单，关键就在于对传输中断点的把握，我就自己的理解画了一个简单的示意图：

![文件断点续传方案](https://img-blog.csdn.net/20141206215834127?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvemhhb3dlbjI1/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

断点续传的关键是断点，所以在制定传输协议的时候要设计好，如上图，我自定义了一个交互协议，每次下载请求都会带上下载的起始点，这样就可以支持从断点下载了，其实HTTP里的断点续传也是这个原理，在HTTP的头里有个可选的字段RANGE，表示下载的范围。

### 6.3 在杭州 HTTP 请求服务器响应快，可能离服务器距离近，而在深圳访问就很慢很慢，会是什么原因？如果用户投诉，怎么分析这个问题？
其实这是一个比较开放的问题，重点在于考察如何排查网络状况：

1. 首先排查是否是本地网络的问题，例如访问其它服务器是否够快，或者测试一下本地的网络；
2. 测试访问网站是否正常，测试DNS解析是否正确，速率如何，丢包是否严重。
3. 检查 CDN 的状态，



 