# http 历史,(来源：[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP))

## 0.9 - 单行协议

    最初的http没有版本号，为了区分与其他的版本号被定位0.9。http/0.9极其简单：请求由单行指令构成，以唯一的可用方法GET开头，其后面跟目标资源路径。连接到服务器后，协议端口号服务器这些都不是必须的。

eg:

request:

```
    GET  /index.html
```

相应也极其简单，只包含响应文档本身。

response:

```
    <HTML>
    这是一个非常简单的HTML页面
    </HTML>
```

    http/0.9版本响应内容并不包含http头，这意味着只有html文件可以传送，无法传输其他类型的文件；也没有状态码错误代码，一个包含问题描述新的HTML文件将被返回以供查看。

## 1.0 - 构建可扩展性

因为 http/0.9 协议的应用十分有限，浏览器和服务器的迅速拓展内容以及用途越来越广。

-   加入了协议版本信息（HTTP/1.0 加入了 GET 行）
-   状态码会在响应开始时发送，使得浏览器能知道请求执行状态，并调整行为。（更新使用 cache）
-   添加了 HTTP 头的概念，无论是请求还是响应，允许传输元数据，使得协议越来越灵活，扩展性也更强。
-   允许传输其他类型的文档能力（http 头：content-type）
    eg:

```
request:
    GET /index.html HTTP/1.0
    User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)
response:
    200 OK
    Date: Tue, 15 Nov 1994 08:12:31 GMT
    Server: CERN/3.0 libwww/2.17
    Content-Type: text/html
    <HTML>
        一个包含图片的页面
        <IMG SRC="/myimage.gif">
    </HTML>
```

```
request:
    GET /myimage.gif HTTP/1.0
    User-Agent: NCSA_Mosaic/2.0 (Windows 3.1)
response:
    200 OK
    Date: Tue, 15 Nov 1994 08:12:32 GMT
    Server: CERN/3.0 libwww/2.17
    Content-Type: text/gif
    (图片内容)
```

## 1.1 – 标准化的协议

此时 http/1.0 并不是官方标准，尽管 1996 年 11 月发表了 RFC 1945，但是他不是官方标准，所以实际使用中显得有些混乱，1997 年初，HTTP/1.1 标准发布，此时 1.0 标准发布也才几个月。HTTP/1.1 在 1997 年 1 月以 RFC 2068 文件发布。

HTTP/1.1 消除了大量歧义内容并引入了多项改进：

-   连接可以复用，节省了多次打开 tcp 连接加载网页文件资源的问题。（HTTP/1.0 默认为每一对 HTTP 请求/响应都打开一个单独的 TCP 连接。）
-   管线化技术，允许第一个应答发送之前就发送第二个请求，以降低通信延迟。
-   相应分块。
-   缓存控制
-   内容协商，包括语言，编码，类型等，允许客户端服务端约定以最合适的内容进行交换。
-   Host 头，不同域名服务器可以配置在同一个 IP 地址服务器上。(Host 请求头指明了请求将要发送到的服务器主机名和端口号。)

## 2.0

HTTP/2 标准于 2015 年 5 月以 RFC 7540 正式发表。 主要基于 [SPDY](https://zh.wikipedia.org/wiki/SPDY) 协议

-   与 HTTP/1.1 在请求方法、状态码乃至 URI 和绝大多数 HTTP 头部字段等方面保持高度兼容性。
-   帧、消息、流和 TCP 连接
    > 有别于 HTTP/1.1 在连接中的明文请求，HTTP/2 与 SPDY 一样，将一个 TCP 连接分为若干个流（Stream），每个流中可以传输若干消息（Message），每个消息由若干最小的二进制帧（Frame）组成。这也是 HTTP/1.1 与 HTTP/2 最大的区别所在。 HTTP/2 中，每个用户的操作行为被分配了一个流编号(stream ID)，这意味着用户与服务端之间创建了一个 TCP 通道；协议将每个请求分割为二进制的控制帧与数据帧部分，以便解析。这个举措在 SPDY 中的实践表明，相比 HTTP/1.1，新页面加载可以加快 11.81% 到 47.7%
-   HPACK 算法，HPACK 算法是新引入 HTTP/2 的一个算法，用于对 HTTP 头部做压缩。
-   服务器推送
-   修复 HTTP/1.0 版本以来未修复的 队头阻塞 问题；
-   对数据传输采用多路复用，让多个请求合并在同一 TCP 连接内。

## 3.0 - 草案状态

此变化主要为了解决 HTTP/2 中存在的队头阻塞问题。受 tcp 拥塞控制的影响，少量丢包就可能影响到整个 tcp 连接上的所有流被阻塞。
基于 [QUIC](https://zh.wikipedia.org/wiki/QUIC)(快速 UDP 网络连接)
