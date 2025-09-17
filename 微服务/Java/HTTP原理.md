## 一、HTTP 1.1

### 1.概述

超文本传输协议 (HTTP) 是一种用于分布式协作超媒体信息系统的应用程序级协议。HTTP 是一种通用的无状态协议，可用于其他目的以及使用其请求方法、错误代码和标头的扩展。

HTTP/1.0 和 HTTP/1.1 之间的主要区别在于 HTTP/1.0 为每个请求/响应交换使用一个新连接，而 HTTP/1.1 连接可用于一个或多个请求/响应交换。

HTTP遵循请求（Request）/响应（Response）模型：

请求从客户端发出，最后服务器端响应该请求并返回。换句话说，肯定是先从客户端开始建立通信的，服务器端在没有接收到请求之前不会发送响应。当客户端向服务器发送个HTTP请求时，请求报文包含请求的方法、URL、协议版本、请求头部和请求数据。

HTTP服务器则在端口（默认是80端口）监听客户端的请求。收到请求后，服务器会返回一个状态行及响应的内容，包括协议的版本、成功或者错误代码、服务器信息、响应头部和响应数据。

### 2.基本功能

三个基本特性使 HTTP 成为一个简单但功能强大的协议：

- **HTTP无连接:**HTTP客户端，即浏览器发起HTTP请求，请求完成后，客户端等待响应。服务器处理请求并发送回响应，然后客户端断开连接。因此客户端和服务器仅在当前请求和响应期间相互了解。进一步的请求是在新连接上发出的，比如客户端和服务器彼此都是新的。
- **HTTP媒体独立:**这意味着，**只要客户端和服务器都知道如何处理数据内容，任何类型的数据都可以通过 HTTP 发送**。客户端和服务器都需要使用适当的 MIME 类型指定内容类型。
- **HTTP无状态:**如上所述，HTTP 是无连接的，它是 HTTP 是无状态协议的直接结果。服务器和客户端仅在当前请求期间相互了解。后来，两个人都忘记了对方。由于协议的这种性质，客户端和浏览器都不能在跨网页的不同请求之间保留信息。

### 3.HTTP请求与响应

HTTP 是一种不保存状态，即无状态（stateless）协议。HTTP 协议自身不对请求和响应之间的通信状态进行保存。也就是说在 HTTP 这个级别，协议对于发送过的请求或响应都不做持久化处理。 每当有新的请求发送时，就会有对应的新响应产生。

这是为了更快地处理大量事务，确保协议的可伸缩性，而特意把 HTTP 协议设 计成如此简单的。

HTTP/1.1 虽然是无状态协议，但为了实现期望的保持状态功能，于是引入了 Cookie 技术。有了 Cookie 再用 HTTP 协议通信，就可以管理状态了。

HTTP协议1.1之前版本的连接都是非持久连接，就是是一个请求一个响应之后，直接就断开了，但是现在的HTTP协议1.1版本不是直接就断开了，而是等一段时间，这段时间是等什么呢，等着用户有后续的操作，如果用户在这段时间之内有新的请求，那么还是通过之前的连接通道来收发消息，如果这段时间用户没有发送新的请求，那么就会断开连接，这样可以提高效率，减少短时间内建立连接的次数，因为建立连接也是耗时的，这个时间是可以通过后端的代码来调整，网站可以根据用户的行为来分析统计出一个最优的等待时间。

主要流程和数据打包过程如下图所示：

![image-20240413120453957](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20240413120453957.png)

![image-20240413120551962](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20240413120551962.png)

请求：

（1）请求DNS服务器，找到URL中域名对应的服务器IP，找到服务器后就可以进行HTTP请求。

（2）客户端生成HTTP请求报文：包括起始行，首部，主体。

（3）建立tcp连接：由于HTTP是基于tcp的应用层协议，所以要此时进行tcp连接：客户端与服务端进行tcp三次握手，建立连接。

（4）发送HTTP请求报文：tcp握手成功后，通过tcp/ip通信协议将客户端生成的HTTP报文，发送至服务器。

响应

（5）服务器生成HTTP响应报文：服务器收到请求后会根据请求内容准备客户端需要的数据，并生成对应的HTTP报文。

（6）发送HTTP响应报文：通过tcp/ip通信协议将服务器生成的HTTP报文，发送至客户端。

（7）tcp断开连接：客户端与服务端进行tcp四次挥手，断开连接，请求/响应结束。

2、在浏览器地址栏键入URL，访问网页本质上也是HTTP请求和响应的过程，此时客户端是浏览器，在整个流程中会有浏览器本身的处理逻辑。在浏览器地址栏键入URL，按下回车之后会经历以下流程： 其中步骤（1）（8）就是浏览器自己的处理逻辑。

（1）浏览器查找缓存：如果查找到缓存中有我们URL对应的网页信息，并且没有过期，如果有则会直接读取缓存内容，此时不会发送HTTP请求，如果没有则发送HTTP请求

（2）请求DNS服务器，找到URL中域名对应的服务器IP，找到服务器后就可以进行HTTP请求。

（3）浏览器生成HTTP请求报文：包括起始行，首部，主体。

（4）建立tcp连接：由于HTTP是基于tcp的应用层协议，所以要此时进行tcp连接：浏览器与服务端进行tcp三次握手，建立连接。

（5）发送HTTP请求报文：tcp握手成功后，通过tcp/ip通信协议将浏览器生成的HTTP报文，发送至服务器。

（6）服务器生成HTTP响应报文：服务器收到请求后会根据请求内容准备浏览器需要的数据，并生成对应的HTTP报文。

（7）发送HTTP响应报文：通过tcp/ip通信协议将服务器生成的HTTP报文，发送至浏览器。

（8）浏览器处理响应内容：浏览器根据返回的结果进行渲染展示，同时判断是否需要将网页信息存入缓存。

（9）tcp断开连接：浏览器与服务端进行tcp四次挥手，断开连接，请求/响应结束。

### 4.请求/响应报文

用于 HTTP 协议交互的信息被称为 HTTP 报文。请求端（客户端）的 HTTP 报文叫做请求报文，响应端（服务器端）的叫做响应报文。 HTTP 报文本身是由多行（用 CR+LF 作换行符）数据构成的字符串文本。

![image-20240413120753563](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20240413120753563.png)

- 起始行（start line）：描述请求或响应的基本信息

- 头部字段集合（header）：使用key-value的形式更详细的说明报文

- 消息正文（entity）：实际传输的数据，它不一定是纯文本，也可以是图片、视频等二进制数据。

前两部分起始行和头部字段经常又合称为“请求头”或“响应头”，消息正文又称为“实体”。

HTTP协议规定报文必须有起始行（start line）和头部字段集合（header），但是可以没有消息正文（entity/body），而且在header之后必须要有一个“空行”，也就是CR+LF。

#### 1.请求报文

![image-20240413121247275](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20240413121247275.png)

请求行：它简要地描述了客户端想要如何操作服务器的资源

- 请求方法：表示对资源的操作方法

- URI：标齐了请求方法要操作的资源

-  协议版本：表示报文使用的HTTP协议版本

请求头部：

- HTTP头部字段非常灵活，可以任意添加自定义字头部字段名。

- 头部字段名不区分大小写，例如“Host”也可以写成“host”，但首字母大写的可读性更好。
- 头部字段名里不允许出现空格，可以使用连字符“-”，但不能使用下划线“_”。例如，“test-name”是合法的头部字段名，而“test name”、“test_name”是不合法的头部字段名。
- 头部字段名后面必须紧接着“:”，不能有空格，而“:”后的字段值前可以有多个空格。
- 头部字段名的顺序是没有意义的，可以任意排列不影响语义。
- 头部字段名原则上不能重复，除非这字段本身的语义允许，例如Set-Cookie。

```http
POST /cgi-bin/process.cgi HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: www.jc2182.com
Content-Type: text/xml; charset=utf-8
Content-Length: length
Accept-Language: en-us
Accept-Encoding: gzip, deflate
Connection: Keep-Moove
<?xml version="1.0" encoding="utf-8"?>
<string xmlns="http://clearforest.com/">string</string>
```

#### 2.响应报文

![image-20240413121618364](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20240413121618364.png)

状态行：服务器响应的状态

- 协议版本：表示报文使用的HTTP协议版本

-  [状态码](https://so.csdn.net/so/search?q=状态码&spm=1001.2101.3001.7020)：一个三位数，用代码的形式表示处理的结果，比如200是成功，500是服务器错误

- 状态码描述：作为数字状态码补充，是更详细的解释文字，帮助人理解原因

响应头部：和“请求头部”相同

响应正文：实际传输的数据，它不一定是纯文本，也可以是图片、视频等二进制数据。

```http
HTTP/1.1 200 OK
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache/2.2.14 (Win32)
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
Content-Length: 88
Content-Type: text/html
Connection: Closed

<html>
<body>
<h1>Hello, World!</h1>
</body>
</html>
```

### 5.URI和URL

URI（Uniform Resource Identifier）：统一资源**标识符**，是一个用于标识某一互联网资源名称的字符串。

URL（Uniform Resource Locator）：统一资源**定位符**，是一个用于标识和定位某一互联网资源名称的字符串。

例如：

- 我们要找一个人——张三，我们可以通过他的唯一的标识来找，比如说身份证，那么这个身份证就唯一的标识了一个人，这个身份证就是一个 URI；

- 而要找到张三，我们不一定要用身份证去找，我们还可以根据地址去找，如在清华大学18号宿舍楼的404房间第一个床铺的张三，我们也可以唯一确定一个张三，住址协议://地球/中国/北京市/清华大学/18号宿舍楼/404号寝/张三.人。而这个地址就是我们用于标识和定位的 URL。

URI 通过任何方法标识一个人即可，而 URL 虽然也可以标识一个人，但是它主要是通过定位地址的方法标识一个人，所以 URL 其实是 URI 的一个子集，即 URL 是靠标识定位地址的一个 URI。组成部分：

```url
scheme:[subscheme]://[username:password@]host:port/path?query#fragment
```

![](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20240413122338892.png)

![](../../../../picbed/store/picbed/img/image-20240413122353939.png)

### 6.常用头部字段

基本上可以分为四大类：

（1）通用字段（general-header）：在请求头和响应头里都可以出现。

（2）请求字段（request-header）：仅能出现在请求头里，进一步说明请求信息或者额外的附加条件。

（3）响应字段（response-header）：仅能出现在响应头里，补充说明响应报文的信息。

（4）实体字段（entity-header）：它实际上属于通用字段，但专门描述body的额外信息（只是描述body信息，字段本身存在于请求头或响应头中）。

**对HTTP报文的解析和处理实际上主要就是对头字段的处理，理解了头字段也就理解了HTTP报文。**

字段详细：https://www.jc2182.com/http/http-header.html

### 7.请求方法

HTTP/1.1协议中共定义了八种方法（有时也叫“动作”），来表明请求指定的资源不同的操作方式。

HTTP1.0定义了三种请求方法： GET，POST 和HEAD方法。

HTTP1.1新增了五种请求方法：OPTIONS，PUT，DELETE，TRACE和CONNECT方法。

#### 1.OPTIONS方法

用来描述了目标资源的通信选项，返回服务器针对特定资源所支持的HTTP请求方法。

特点：请求没有请求体

![image-20240413123358212](../../../../picbed/store/picbed/img/image-20240413123358212.png)

响应：响应头部中的“Allow”就是特定资源支持的HTTP请求方法

![image-20240413123423854](../../../../picbed/store/picbed/img/image-20240413123423854.png)

#### 2.GET方法

GET方法只是用来获取数据，不对服务器的数据做任何的修改，新增，删除等操作。生成的数据应作为响应中的实体（entry/body）。

GET请求会把**请求的参数附加在URL后面，没有请求体，这样是不安全的**，在处理敏感数据时最好对参数做加密处理，或者使用其它请求方法处理敏感数据。并且URL后面只能携带字符串，不能携带图片、视频或其它类型的数据。

HTTP协议并没有限制GET方法对URL大小，但是不同的浏览器对其有不同的大小长度限制。

请求的过程：

- 浏览器请求TCP连接（第一次握手）
- 服务器答应进行TCP连接（第二次握手）
- 浏览器确认，并发送GET请求头和数据（第三次握手，这个报文比较小，所以HTTP会在此时进行第一次数据发送）
- 服务器返回200 OK响应

特点：

- 请求：没有请求体

![image-20240413123618627](../../../../picbed/store/picbed/img/image-20240413123618627.png)

- 响应

  ![image-20240413123643916](../../../../picbed/store/picbed/img/image-20240413123643916.png)

#### 3.HEAD方法

HEAD方法与GET方法相同，只是HEAD的响应中没有响应正文，**只有起始行和响应头**，而且**同样的请求内容使用HEAD方法和GET方法请求，响应的起始行和响应头是完全相同的**。这种方法是**通常用于测试超文本连接的有效性、可访问性，以及最近的修改**。

特点：

- 请求：没有请求体
- 响应：没有响应正文

#### 4.POST方法

向指定资源**提交数据进行处理请求**（例如提交表单或者上传文件）。**数据被包含在请求体中**。POST请求可能会导致新的资源的建立和/或已有资源的修改。

请求过程：

（1）浏览器请求TCP连接（第一次握手）
（2）服务器答应进行TCP连接（第二次握手）
（3）浏览器确认，并发送POST请求头（第三次握手，这个报文比较小，所以http会在此时进行第一次数据发送）同时可以确定源服务器是否愿意接受该请求（基于请求头）
（4）服务器返回100 Continue响应
（5）浏览器发送请求体数据
（6）服务器返回200 OK响应

特点：

- 请求

  ![image-20240413123926517](../../../../picbed/store/picbed/img/image-20240413123926517.png)

- 响应

  ![image-20240413123949280](../../../../picbed/store/picbed/img/image-20240413123949280.png)

#### 5.PUT方法

数据发送到服务器以创建或更新资源，**侧重于创建数据**。

特点：

- 请求

  ![image-20240413124102604](../../../../picbed/store/picbed/img/image-20240413124102604.png)

- 响应

  ![image-20240413124117273](../../../../picbed/store/picbed/img/image-20240413124117273.png)

#### 6.DELETE方法

DELETE方法请求源服务器删除由URI标识的资源。

- 请求

  ![image-20240413124200219](../../../../picbed/store/picbed/img/image-20240413124200219.png)

- 响应

  ![image-20240413124216217](../../../../picbed/store/picbed/img/image-20240413124216217.png)

#### 7.TRACE方法

TRACE方法用于沿着目标资源的路径执行消息环回，回显服务器收到的请求，以便客户可以看到中间服务器进行了哪些（假设任何）进度或增量。**主要用于测试或诊断**。

- 请求

  ![image-20240413124315148](../../../../picbed/store/picbed/img/image-20240413124315148.png)

- 响应

  ![image-20240413124333080](../../../../picbed/store/picbed/img/image-20240413124333080.png)

#### 8.CONNECT方法

HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。（例如SSL编码的通信）

### 8.GET方法和POST方法的区别

1、参数传递的方式不同，**GET请求的参数是直接拼接在地址栏URL的后面，而POST请求的参数是放到请求体里面的**。

2、**作用不同**和**使用场景不同**，POST用于修改和写入数据，GET一般用于搜索排序和筛选之类的操作（淘宝，支付宝的搜索查询都是get提交），目的是资源的获取，读取数据。

3、POST的**安全性**更好，因为GET请求的数据都是明文显示在URL上面的，所以安全和私密性不如POST。

4、POST能发送更多的**数据类型**而GET只能发送字符，因为POST的请求数据是放在请求体里的数据可以是字符、图片甚至是音视频数据，而GET的请求数据是放在URL后面，所以请求数据只能是字符类型。

5、**POST的请求速度比GET慢，因为GET产生一个数据包，POST产生两个数据包**。对于GET请求，浏览器会把http header 和 请求数据 一并发出去，服务器响应返回数据。而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送请求数据，服务器响应返回数据。所以POST发送了两次数据，而GET只发送一次数据。

6、浏览器会将GET会将数据缓存起来，而POST不会。

### 9.状态码

状态码存在于响应报文的状态行中，状态代码旨在给机器使用，而状态码对应的含义是为人而设计。共有5类：

![image-20240413124611914](../../../../picbed/store/picbed/img/image-20240413124611914.png)

常见的状态码：

![image-20240413124640347](../../../../picbed/store/picbed/img/image-20240413124640347.png)

### 10.HTTP常用技术

#### 1.数据统计与防盗链

1、HTTP请求头里，有一个**referer字段**，内容是上一页的地址，即可以告知服务端当前的请求的来源。比如一篇CSND的文章（连接为A）中引用了一个知乎的连接（连接为B），点击这个链接后，就会在浏览器在请求头里会将referer设置为CSDN文章的链接A，当知乎收到请求后就知道这次的流量是从CSDN过来的。这也就是为什么服务器知道我们请求是在哪发起的。但是如果直接从浏览器里直接输入地址，回车进去，此时没有referer信息。

2、针对此特点衍生出两个作用：

（1）数据统计：这对于数据分析来说，是非常重要的，比如，你在许多网站上投放了广告，你需要知道每个网站上的点击量，用户点击后来到你的网站，你就可以通过referer这个首部来统计各个网站的点击情况了，进而优化广告投放方法。

（2）防盗链：如果A网站引用了B网站的图片，从A网站访问该图片时，referer头信息就是A网站的链接，此时B网站收到信息后就可以知道本次请求是从A网站过来的，如果此图片仅允许在B网站内访问，不允许从其它网站访问则B网站可以不返回图片信息，此时A网站就无法引用该图片。A网站可能出现如下所图所示的情况。这就是利用referer做的防盗链。

#### 2.状态管理

HTTP协议虽然是一个无状态协议，但是有了Cookie技术就可以管理状态了。Cookie技术通过在请求和响应报文中写入Cookie信息来控制客户端的状态。Cookie就是一个本文信息，它可以用来做用户认证，服务器校验等通过文本数据可以处理的问题。

Cookie是服务器生成的，通过HTTP响应发送给客户端，由客户端保存起来：Cookie会根据从服务器端发送的响应报文内的一个叫做Set-Cookie的首部字段信息，通知客户端保存Cookie，将Cookie保存在用户浏览器端的。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入Cookie值后发送出去。服务器端发现客户端发送过来的Cookie后，会去检查究竟是从哪一个客户端发来的连接请求，然后对比服务器上的记录，最后得到之前的状态信息。

![image-20240413125009148](../../../../picbed/store/picbed/img/image-20240413125009148.png)

没有Cookie时，每次访问都像第一次访问一样，无法判断用户是否访问过，每次访问都需要重新填写用户信息。有Cookie后，仅第一次访问需要填写用户信息，后续访问可以发送Cookie，不用填写用户信息即可登录。

#### 3.HTTP缓存

浏览器缓存是一种在本地保存资源副本，以供下次请求时直接使用的技术。

当浏览器在真正发起网络请求之前，浏览器会先在浏览器缓存中查询是否有要请求的文件。如果浏览器发现请求资源已经存在浏览器缓存中存有副本，则会拦截请求并返回该资源副本结束请求。如果查找缓存失败，则会进入网络请求。这样可以减轻的服务器负担，同时也节省了通信流量和通信时间。

即便浏览器内有缓存，也不能保证每次都会返回对同资源的请求。因为这关系到被缓存资源的有效性问题。 当遇上服务器上的资源更新时，如果还是使用不变的缓存，那就会 演变成返回更新前的“旧”资源了。 即使存在缓存，也会因为客户端的要求、缓存的有效期等因素，向服务器确认资源的有效性。若判断缓存失效，缓存服务器将会再次从服务器上获取“新”资源。

浏览器是通过响应头中的Cache-Control字段来设置是否缓存该资源。通常，我们还需要为这个资源设置一个缓存过期时长，而这个时长是通过Cache-Control中的Max-age参数来设置的。因此在该缓存资源还未过期的情况下, 如果再次请求该资源，会直接返回缓存中的资源给浏览器。如果缓存过期了，浏览器则会继续发起网络请求，并且在HTTP请求头中带上If-None-Match、If-Modified-Since，服务器收到请求头后，会根据If-Modified-Since、If-None-Match以及Etag的值来判断请求的资源是否有更新。如果没有更新，就返回304 Not Modified，相当于服务器告诉浏览器，这个缓存可以继续使用。如果资源有更新，服务器就直接返回最新资源给浏览器。

整体的流程可以参考下图，先查找缓存然后再根据情况看是否发起DNS解析进而发送HTTP请求。

![image-20240413125212473](../../../../picbed/store/picbed/img/image-20240413125212473.png)

请求过程：

（1）第一次请求：返回200 OK，请求头：没有任何特殊， 响应头：

![](../../../../picbed/store/picbed/img/image-20240413125406716.png)

Etag可以理解成文件的唯一标识，Last-Modified是文件的最后修改时间，浏览器会记录这两个信息，下次请求时会放在请求头中，服务器就可以根据请求头中的这两个信息来判断浏览器缓存的文件和服务器中的文件是否相同。如果相同则服务器会返回304 Not Modified表示文件没有修改，此时没有响应体，浏览器收到304后会从本地缓存中获取响应内容。如果不相同则服务器会返回200 OK，响应体中就是浏览器请求的文件内容。

（2）第二次请求：返回304 Not Modified

![image-20240413125449563](../../../../picbed/store/picbed/img/image-20240413125449563.png)

请求头：

![image-20240413125508432](../../../../picbed/store/picbed/img/image-20240413125508432.png)

请求头中的Cache-Control浏览器设置成0，所以此时浏览器不会先查找本地缓存而是会直接发送HTTP请求至服务器。

**两个条件满足一个服务器就会返回全部文件内容**：

（1）如果自If-Modified-Since这个时间点以后，请求的图片修改过，则不使用浏览器缓存，服务器要返回文件内容。

（2）如果文件最新的Etag的值和If-None-Match的值不匹配，则不使用缓存，服务器要返回文件内容。

响应头：

![image-20240413125601078](../../../../picbed/store/picbed/img/image-20240413125601078.png)

服务器获取到文件的最新的Etag，和文件的最后修改时间Last-Modified。

综合请求头和响应头的信息来看，响应头的Last-Modified的值没有超过If-Modified-Since的值，并且响应头的Etag和请求头的If-None-Match相同，所以服务器判断客户端使用本地缓存即可，因为客户端的本地缓存的文件和服务器的保存的文件是相同的，此时返回304 Not Modified，此时服务器不会返回文件内容，也就是没有响应体，提高了响应速度。



#### 4.HTTP内容压缩

为了提高网页在网络上的传输速度、节省流量，服务器会对响应体信息进行压缩，压缩方法如常见的gzip压缩、deflate压缩、compress压缩、sdch压缩等。能大大减少网络传输的数据量，提高了用户显示网页的速度。

压缩的过程：

（1）浏览器发送HTTP请求给服务器,  请求头中有Accept-Encoding表示浏览器支持什么样的压缩方式。

（2）服务器接到请求后，生成原始的响应体, 其中有原始的Content-Type和Content-Length。

（3）服务器对响应体进行压缩编码， 编码响应头中有压缩后的Content-Type和Content-Length(压缩后的大小)， 并且增加了Content-Encoding表示压缩方法，然后把响应发送给浏览器。

（4）浏览器接到响应后，根据Content-Encoding表示的压缩方法来对响应体进行解码。 获取到原始响应体后， 再显示出网页。

请求过程：

请求头：说明浏览器支持gzip、deflate、br三种压缩方式。

![image-20240413125744961](../../../../picbed/store/picbed/img/image-20240413125744961.png)

响应头：说明响应体是gzip压缩方式，浏览器浏览器会根据这种方式解压。

![image-20240413125805000](../../../../picbed/store/picbed/img/image-20240413125805000.png)



#### 5.持久连接与分块传输

有的时候，服务器无法确定HTTP回应的消息的大小，比如非常大的文件的下载，或者处理的逻辑比较复杂，需要一边处理一边实时生成消息，这个时候服务器一般都使用分块chunked编码。此时，服务器不会带上Content-Length这个响应头字段，带上了另外一个响应头字段：Transfer-encoding:chunked。如果报文中包含Transfer-encoding:chunked首部, 那么Content-Length将被忽略。

chunked编码使用若干个chunk组成，由一个标明长度为0的chunk结束，每个Chunk有两部分组成，每个部分用回车换行隔开。因此，接收数据的时候，需要首先获取每一个chunk数据的字节长度，然后，跳过2个字节(/r/n)，取出数据，然后，再跳过2个字节(/r/n)，获取下一个chunk的长度，直到最后一个chunk，最后一个chunk一定是0，并且字节的长度都是十六进制形式传输，需要进行相应的转化成十进制，如果是gzip格式的数据，那么，在最后完成所有数据组合之后，需要再解压。

比如传输内容：先读取123字节内容，然后再读区41字节内容，最后读到0H（也就是一个空包），读取结束，即最后一个块的大小为0，用于表示数据传输结束。

123H/r/n

....（123字节内容）

41H/r/n

...（41字节内容）

0H/r/n（结束）

分块传输优点：

（1）动态内容，Content-length无法预知：HTTP分块传输编码允许服务器为动态生成的内容维持HTTP持久连接。通常，持久连接需要服务器在开始发送消息体前发送Content-Length消息头字段，但是对于动态生成的内容来说，在内容创建完之前是不可知的。

（2）gzip压缩，压缩与传输同时进行：HTTP服务器有时使用压缩 （gzip或deflate）以缩短传输花费的时间。分块传输编码可以用来分隔压缩对象的多个部分。在这种情况下，块不是分别压缩的，而是整个负载进行压缩，压缩的输出使用本文描述的方案进行分块传输。在压缩的情形中，分块编码有利于一边进行压缩一边发送数据，压缩过程中，压缩完成的内容可以分块传输，未压缩的部分可以继续压缩，而不是先完成所有压缩过程以得知压缩后数据的大小再分块传输。

（3）散列签名，需缓冲完成才能计算：分块传输编码允许服务器在最后发送消息头字段。对于那些头字段值在内容被生成之前无法知道的情形非常重要，例如消息的内容要使用散列进行签名，散列的结果通过HTTP消息头字段进行传输。没有分块传输编码时，服务器必须缓冲内容直到完成后计算头字段的值并在发送内容前发送这些头字段的值。



### 11.HTTP使用场景及优缺点

#### 1.使用场景

（1）Web应用，HTTP最广泛的应用就是Web应用程序。无论是桌面端的浏览器还是移动端的应用程序，HTTP都是应用层协议。

（2）API接口，在Web应用程序中，API接口是连接前端UI和后端数据的桥梁。HTTP协议的接口设计，可以使不同语言、不同框架的应用程序在接口层面得到统一，以方便数据的交互与共享。 

#### 2.优点

（1）简单易用：HTTP 协议的语法和规范相对简单，容易学习和使用。

（2）可扩展性：HTTP 协议支持插件和扩展，可以根据需要添加新的功能和特性。

（3）传输超文本：HTTP 协议是传输超文本的标准协议，可以在网页中嵌入各种形式的媒体内容。

（4）分布式：HTTP 协议是基于客户端-服务器模式的，可以支持分布式计算和资源共享。

（5）跨平台性：HTTP 协议是跨平台的，可以在不同的操作系统、编程语言和硬件平台上使用，具有较好的兼容性。

（6）可读性强：HTTP 协议使用文本形式来表示请求和响应，具有较好的可读性，方便调试和排错。

（7）支持多种传输方式：HTTP 协议支持多种传输方式，如普通文本、JSON、XML 等，可以适应不同的应用场景。

#### 2.缺点

（1）安全性差：HTTP 协议是明文传输的，数据容易被窃听和篡改，因此安全性较差，需要额外的安全机制来保护数据的安全。

（2）性能较低：HTTP 协议在传输大量数据和处理高并发请求时，性能较低，容易导致网络拥塞和延迟。

（3）不支持推送功能：HTTP 协议不支持服务器向客户端主动推送数据的功能，客户端需要定期向服务器发送请求才能获取最新的数据。

（4）没有优先级控制：HTTP 协议没有优先级控制的机制，所有的请求和响应都被视为同等重要，这可能会影响一些特定应用场景的性能表现。

（5）请求-响应模式：HTTP 协议采用请求-响应模式，即客户端必须等待服务器响应后才能进行下一步操作，这可能会影响用户体验和应用程序的响应速度。   

## 二、RESTFul风格设计和实战

### 2.1 RESTFul风格概述

#### 2.1.1 RESTFul风格简介

RESTful（Representational State Transfer）是一种软件架构风格，用于设计网络应用程序和服务之间的通信。它是一种基于标准 HTTP 方法的简单和轻量级的通信协议，广泛应用于现代的Web服务开发。

通过遵循 RESTful 架构的设计原则，可以构建出易于理解、可扩展、松耦合和可重用的 Web 服务。RESTful API 的特点是简单、清晰，并且易于使用和理解，它们使用标准的 HTTP 方法和状态码进行通信，不需要额外的协议和中间件。

总而言之，RESTful 是一种基于 HTTP 和标准化的设计原则的软件架构风格，用于设计和实现可靠、可扩展和易于集成的 Web 服务和应用程序！

![](../../../../picbed/store/picbed/img/image_X8M-XfzI_A.png)

学习RESTful设计原则可以帮助我们更好去设计HTTP协议的API接口！！

#### 2.1.2 RESTFul风格特点

1.  每一个URI代表1种资源（URI 是名词）；
2.  客户端使用GET、POST、PUT、DELETE 4个表示操作方式的动词对服务端资源进行操作：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源；
3.  资源的表现形式是XML或者**JSON**；
4.  客户端与服务端之间的交互在请求之间是无状态的，从客户端到服务端的每个请求都必须包含理解请求所必需的信息。

#### 2.1.3 **RESTFul风格设计规范**

##### 1.**HTTP协议请求方式要求**

REST 风格主张在项目设计、开发过程中，具体的操作符合**HTTP协议定义的请求方式的语义**。

| 操作     | 请求方式 |
| -------- | -------- |
| 查询操作 | GET      |
| 保存操作 | POST     |
| 删除操作 | DELETE   |
| 更新操作 | PUT      |

##### 2.**URL路径风格要求**

REST风格下每个资源都应该有一个唯一的标识符，例如一个 URI（统一资源标识符）或者一个 URL（统一资源定位符）。资源的标识符应该能明确地说明该资源的信息，同时也应该是可被理解和解释的！

使用URL+请求方式确定具体的动作，他也是一种标准的HTTP协议请求！

- 使用名词表示资源，使用HTTP Method 描述操作。**因为"资源"表示一种实体，所以应该是名词，URL不应该有动词，动词应该放在HTTP协议中，CRUD的操作不要体现在URL中。**

- #### 资源既可以是单个，也可以是一个集合，比如：user是单个资源，users是集合资源。 由于英文名词的复数规则众多，为了保持接口命名的一致性，我们建议URL里描述集合资源的名词统一使用单数。

| 操作 | 传统风格                | REST 风格                                  |
| ---- | ----------------------- | ------------------------------------------ |
| 保存 | /CRUD/saveEmp           | URL 地址：/CRUD/emp&#xA;请求方式：POST     |
| 删除 | /CRUD/removeEmp?empId=2 | URL 地址：/CRUD/emp/2&#xA;请求方式：DELETE |
| 更新 | /CRUD/updateEmp         | URL 地址：/CRUD/emp&#xA;请求方式：PUT      |
| 查询 | /CRUD/editEmp?empId=2   | URL 地址：/CRUD/emp/2&#xA;请求方式：GET    |

##### 3.**版本号**

应该将API的版本号放入URL。 版本号以字符'v'开头，比如：`v1`

```java
http://demo.com/cuc/sever/v1
```

##### 4.**使用小写字母，多个单词用"-"分隔，提高URL的可读性**

**RFC3986** 定义了URI是大小写敏感的(除了scheme和host部分)，为了避免歧义，接口路径应尽量使用小写字母。

推荐使用'-'(hyphen)分隔，避免使用驼峰命名（camelCase）、下划线命名(snake_case)、首字母大写的驼峰命名（PascalCase），以提高可读性。

```apl
camelCase  :  /v1/customer/1000/shippingAddress  [Bad]
snake_case :  /v1/customer/1000/shipping_address [Bad]
PascalCase :  /v1/customer/1000/ShippingAddress  [Bad]
 
hyphen     :  /v1/customer/1000/shipping-address [OK]
```

##### 5.**资源嵌套层次避免过深，尽量不超过2层**

定义REST接口时，通常使用资源嵌套（多级路径）来标识资源之间的关系，资源嵌套层次尽量不超过2层，否则很难阅读和使用。

```api
GET  /v1/user/1000/company    查询ID等于1000的用户的公司信息
GET  /v1/user/1000/company/10 查询ID等于1000的用户所属公司中ID=10的公司信息，嵌套两层
```

##### 6.筛选(Filtering)、排序(Sorting)、分页(Pagination)、字段选择

- 筛选(Filtering):接口URL尽量保持简短，对于集合资源不同纬度的筛选，可通过组合不同的Query Param来实现。

  ```api
  // bad
  GET /v1/externalEmployees
  GET /v1/internalEmployees
  GET /v1/internalAndSeniorEmployees
  // good
  GET /v1/employee?state=internal&title=senior
  ```

- 排序(Sorting):   多个字段排序：字段名前缀,`'+'`：表示升序，`'-'`：降序，多个字段名以逗号分隔

  ```api
  GET /v1/employee?sort=age&order=asc
  GET /v1/employee?sort=age&order=desc
  
  GET /v1/employee?sort=+age,-userName
  ```

- 分页(Pagination): 常规分页有两个重要的参数：

  - pageNo 获取那一页的记录，默认第一页。

  - pageSize 每页返回多少条数据，推荐默认值25，必须限定此参数的最大值。

  - 返回的资源列表为：[(pageNo-1)*pageSize, pageNo*pageSize)

    ```
    GET /v1/employee?pageNo=8&pageSize=20&sort=age&order=desc
    ```

- 字段选择: 调用方可能只需要资源的少量属性，接口提供方可支持客户端通过请求参数`fields`来选择返回的字段，多个字段以逗号分隔

  ```
  GET /v1/employee?fields=id,name,age&pageNo=1&pageSize=20&sort=age&order=desc
  ```

  



#### 2.1.4 RESTFul风格好处

1. 含蓄，安全

   使用问号键值对的方式给服务器传递数据太明显，容易被人利用来对系统进行破坏。使用 REST 风格携带数据不再需要明显的暴露数据的名称。

2. 风格统一

   URL 地址整体格式统一，从前到后始终都使用斜杠划分各个单词，用简单一致的格式表达语义。

3. 无状态

   在调用一个接口（访问、操作资源）的时候，可以不用考虑上下文，不用考虑当前状态，极大的降低了系统设计的复杂度。

4. 严谨，规范

   严格按照 HTTP1.1 协议中定义的请求方式本身的语义进行操作。

5. 简洁，优雅

   过去做增删改查操作需要设计4个不同的URL，现在一个就够了。

   | 操作 | 传统风格                | REST 风格                                  |
   | ---- | ----------------------- | ------------------------------------------ |
   | 保存 | /CRUD/saveEmp           | URL 地址：/CRUD/emp&#xA;请求方式：POST     |
   | 删除 | /CRUD/removeEmp?empId=2 | URL 地址：/CRUD/emp/2&#xA;请求方式：DELETE |
   | 更新 | /CRUD/updateEmp         | URL 地址：/CRUD/emp&#xA;请求方式：PUT      |
   | 查询 | /CRUD/editEmp?empId=2   | URL 地址：/CRUD/emp/2&#xA;请求方式：GET    |

6. 丰富的语义

   通过 URL 地址就可以知道资源之间的关系。它能够把一句话中的很多单词用斜杠连起来，反过来说就是可以在 URL 地址中用一句话来充分表达语义。

   > [http://localhost:8080/shop](http://localhost:8080/shop "http://localhost:8080/shop") [http://localhost:8080/shop/product](http://localhost:8080/shop/product "http://localhost:8080/shop/product") [http://localhost:8080/shop/product/cellPhone](http://localhost:8080/shop/product/cellPhone "http://localhost:8080/shop/product/cellPhone") [http://localhost:8080/shop/product/cellPhone/iPhone](http://localhost:8080/shop/product/cellPhone/iPhone "http://localhost:8080/shop/product/cellPhone/iPhone")

### 4.2 RESTFul风格实战

#### 2.2.1 需求分析

-   数据结构： User {id 唯一标识,name 用户名，age 用户年龄}
-   功能分析
    -   用户数据分页展示功能（条件：page 页数 默认1，size 每页数量 默认 10）
    -   保存用户功能
    -   根据用户id查询用户详情功能
    -   根据用户id更新用户数据功能
    -   根据用户id删除用户数据功能
    -   多条件模糊查询用户功能（条件：keyword 模糊关键字，page 页数 默认1，size 每页数量 默认 10）

#### 2.2.2 RESTFul风格接口设计

1. **接口设计**

   | 功能     | 接口和请求方式   | 请求参数                        | 返回值       |
   | -------- | ---------------- | ------------------------------- | ------------ |
   | 分页查询 | GET  /user       | page=1\&size=10                 | { 响应数据 } |
   | 用户添加 | POST /user       | { user 数据 }                   | {响应数据}   |
   | 用户详情 | GET /user/1      | 路径参数                        | {响应数据}   |
   | 用户更新 | PUT /user        | { user 更新数据}                | {响应数据}   |
   | 用户删除 | DELETE /user/1   | 路径参数                        | {响应数据}   |
   | 条件模糊 | GET /user/search | page=1\&size=10\&keywork=关键字 | {响应数据}   |

2. **问题讨论**

   为什么查询用户详情，就使用路径传递参数，多条件模糊查询，就使用请求参数传递？

   误区：restful风格下，不是所有请求参数都是路径传递！可以使用其他方式传递！

   在 RESTful API 的设计中，路径和请求参数和请求体都是用来向服务器传递信息的方式。

   -   对于查询用户详情，使用路径传递参数是因为这是一个单一资源的查询，即查询一条用户记录。使用路径参数可以明确指定所请求的资源，便于服务器定位并返回对应的资源，也符合 RESTful 风格的要求。
   -   而对于多条件模糊查询，使用请求参数传递参数是因为这是一个资源集合的查询，即查询多条用户记录。使用请求参数可以通过组合不同参数来限制查询结果，路径参数的组合和排列可能会很多，不如使用请求参数更加灵活和简洁。
       此外，还有一些通用的原则可以遵循：
   -   路径参数应该用于指定资源的唯一标识或者 ID，而请求参数应该用于指定查询条件或者操作参数。
   -   请求参数应该限制在 10 个以内，过多的请求参数可能导致接口难以维护和使用。
   -   对于敏感信息，最好使用 POST 和请求体来传递参数。

#### 2.2.3 后台接口实现

准备用户实体类：

```java
package com.atguigu.pojo;

/**
 * projectName: com.atguigu.pojo
 * 用户实体类
 */
public class User {

    private Integer id;
    private String name;

    private Integer age;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```

准备用户Controller:

```java
/**
 * projectName: com.atguigu.controller
 *
 * description: 用户模块的控制器
 */
@RequestMapping("user")
@RestController
public class UserController {

    /**
     * 模拟分页查询业务接口
     */
    @GetMapping
    public Object queryPage(@RequestParam(name = "page",required = false,defaultValue = "1")int page,
                            @RequestParam(name = "size",required = false,defaultValue = "10")int size){
        System.out.println("page = " + page + ", size = " + size);
        System.out.println("分页查询业务!");
        return "{'status':'ok'}";
    }


    /**
     * 模拟用户保存业务接口
     */
    @PostMapping
    public Object saveUser(@RequestBody User user){
        System.out.println("user = " + user);
        System.out.println("用户保存业务!");
        return "{'status':'ok'}";
    }

    /**
     * 模拟用户详情业务接口
     */
    @PostMapping("/{id}")
    public Object detailUser(@PathVariable Integer id){
        System.out.println("id = " + id);
        System.out.println("用户详情业务!");
        return "{'status':'ok'}";
    }


    /**
     * 模拟用户更新业务接口
     */
    @PutMapping
    public Object updateUser(@RequestBody User user){
        System.out.println("user = " + user);
        System.out.println("用户更新业务!");
        return "{'status':'ok'}";
    }


    /**
     * 模拟条件分页查询业务接口
     */
    @GetMapping("search")
    public Object queryPage(@RequestParam(name = "page",required = false,defaultValue = "1")int page,
                            @RequestParam(name = "size",required = false,defaultValue = "10")int size,
                            @RequestParam(name = "keyword",required= false)String keyword){
        System.out.println("page = " + page + ", size = " + size + ", keyword = " + keyword);
        System.out.println("条件分页查询业务!");
        return "{'status':'ok'}";
    }
}
```

## 
