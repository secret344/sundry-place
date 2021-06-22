# 跨源资源共享（CORS）

基于 HTTP 头部的机制，允许服务器标示除了他自己以外的 origin（域，协议，端口）请求，可以访问加载资源。跨源资源共享还通过一种机制来检查服务器是否会允许要发送的真实请求，跨域资源共享会通过浏览器发起预检请求来检查。预检请求会标识真实 HTTP 方法以及真实请求中会用到的头。

## 案例

-   使用 XMLHttpRequest 或者 Fetch 发起的跨源 HTTP 请求。
-   Web 字体（css 中@font-face 使用跨源字体资源），因此网站就可以发布 TrueType 字体资源，只允许授权网站使用。
-   WebGl 贴图
-   使用 drawImage 将 Images/video 画面绘制到 Canvas

## 概述

新增了头部来允许服务器声明哪些源站通过浏览器有权访问哪些资源。另外，对那些可能对服务器产生副作用的 HTTP 方法（除 GET 以外，以及搭配某些 MIME 的 POST 请求。），浏览器必须先发送 OPTIONS 方法发起预检请求，从而得知服务器是否允许跨源请求。服务器确认允许后，才能发起实际的 HTTP 请求。在预检请求中，服务端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据。）。

CORS 请求失败会产生错误，为了安全只能查看浏览器控制台来得知错误信息。

## 简单请求 非简单请求

### 简单请求

某些请求不会触发预检请求，称之为简单请求。

1.  使用下列请求方法:
    -   GET
    -   POST
    -   HEAD
2.  除了用户代理自动设置的字段（例如 Connection，User-Agent）和 Fetch 规范中定义的[禁用首部名称](https://fetch.spec.whatwg.org/#forbidden-header-name)的其他首部，允许人为设置的字段为 Fetch 规范定义的 [ 对 CORS 安全的首部字段集合](https://fetch.spec.whatwg.org/#cors-safelisted-request-header)，该集合为：
    -   Accept
    -   Accept-Language
    -   Content-Language
    -   Content-Type(额外限制)
        -   text/plain
        -   multipart/form-data
        -   application/x-www-form-urlencoded
    -   DPR
    -   Downlink
    -   Save-Date
    -   Viewport-Width
    -   Width
    -   请求中的任意 XMLHttpRequestUpload 对象均没有注册任何事件监听器；XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问。
    -   请求中没有使用 ReadableStream 对象。

> WebKit Nightly 和 Safari Technology Preview 为 Accept, Accept-Language, 和 Content-Language 首部字段的值添加了额外的限制。如果这些首部字段的值是“非标准”的，WebKit/Safari 就不会将这些请求视为“简单请求”。

> 这些跨站点请求与浏览器发出的其他跨站点请求并无二致。如果服务器未返回正确的响应首部，则请求方不会收到任何数据。因此，那些不允许跨站点请求的网站无需为这一新的 HTTP 访问控制特性担心。

```
    假如站点 http://foo.example 的网页应用想要访问 http://bar.other 的资源。
    GET /resource/public-data/ HTTP/1.1
    Host: bar.other
    User-Agent: Mozilla/...
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-us;es;q=0.5
    Accept-Encoding: gzip,deflate
    Accept-Charset: ISO-8859-1;utf-8;q=0.7,*;q=0.7
    Connection: keep-alive
    Referer: http://foo.example/examples/access-control/simpleXSInvocation.html
    Origin: http://foo.example
    请求体,
    1：请求方法  目标资源路径 协议版本号
    2：Host 请求将要发送到的服务器主机名和端口号
    3：用户代理软件的应用类型、操作系统、软件开发商以及版本号。
    4：告知（服务器）客户端可以处理的内容类型
    5：客户端声明它可以理解的自然语言
    6：客户端声明它可以理解的编码方式
    7：客户端可以处理的字符集类型
    8：决定当前的事务完成后，是否会关闭网络连接
    9：包含了当前请求页面的来源页面的地址
    10：请求来自于哪个站点

    response:
    HTTP/1.1 200 OK
    Date: Mon, 01 Dec 2008 00:23:22 GMT
    Server: Apache/2.0.61
    Access-Control-Allow-Origin: *
    Keep-Alive: timeout=2,max=100
    Transfer-Encoding: chunked
    Content-Type: application/xml

    [xml Data]
    响应体,
    1：协议版本号 状态码  状态信息
    2：时间
    3：源头服务器所用到的软件相关信息
    4：该响应的资源是否被允许与给定的origin共享
    5：允许消息发送者暗示连接的状态，还可以用来设置超时时长和最大请求数
    6：将 [entity] 安全传递给用户所采用的编码形式
    7：指示资源的MIME类型
```

[entity](https://developer.mozilla.org/zh-CN/docs/Glossary/Entity_header)

上述为一个简单请求的流程，服务器检查请求体中的 Origin 字段，来判断是否同意这次请求。

上文中响应体返回 Access-Control-Allow-Origin: \* 代表允许任意外域访问。如果服务端仅允许 http://foo.example 访问，则修改如下：

```
Access-Control-Allow-Origin: http://foo.example
```

现在外域就不能访问了。

### 预检请求(非简单请求)

与简单请求不同，预检请求要求必须首先使用 OPTIONS 发起一个预检请求到服务器，以获知服务器是否允许该实际请求。预检请求的使用，可以避免跨域请求对用户数据产生未预期的影响。

不写例子了。。。。。。推荐[MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS)

浏览器在检测到在 js 发送的请求需要预检，就会使用 OPTIONS 方法发送预检请求，OPTIONS 是 HTTP/1.1 协议中定义的方法，用以从服务器获取更多信息。该方法不会对服务器资源产生影响。

大多数浏览器不支持针对于预检请求的重定向。如果一个预检请求发生了重定向，浏览器将报告错误。可以通过先发送简单请求，获取真实地址，在发出另外一个复杂请求。如果请求是由于存在 Authorization 字段而引发了预检请求，则这一方法将无法使用。这种情况只能由服务端进行更改。

### 附带身份凭证的请求

一般而言，对于跨源 XMLHttpRequest 或 Fetch 请求，浏览器不会发送身份凭证信息。如果要发送凭证信息，需要设置 XMLHttpRequest 的某个特殊标志位。

本例中，http://foo.example 的某脚本向 http://bar.other 发起一个 GET 请求，并设置 Cookies：

```
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/credentialed-content/';

function callOtherDomain(){
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.withCredentials = true;
    invocation.onreadystatechange = handler;
    invocation.send();
  }
}
```

将 XMLHttpRequset 的 withCredentials 设置为 true，从而向服务器发送 Cookies。因为是一个简单 GET 请求，所以不会发生预检请求。但是服务器响应如果不存在 Access-Control-Allow-Credentials: true ，浏览器将不会把响应内容返回给请求的发送者。

### 附带身份凭证的请求与通配符

对于附带身份凭证的请求，服务器不得设置 Access-Control-Allow-Origin 的值为“\*”。

请求首部携带 Cookie 信息，如果 Access-Control-Allow-Origin 的值为“\*”，请求将会失败。而将 Access-Control-Allow-Origin 的值设置为 http://foo.example，则请求将成功执行。

另外，响应首部中携带了 Set-Cookie 字段，尝试对 Cookie 进行修改。如果操作失败，将会抛出异常。

> 注意在 CORS 响应中设置的 cookies 适用一般性第三方 cookie 策略。比如，页面是在 `foo.example` 加载，但是 cookie 是被 `bar.other` 发送的，如果用户设置其浏览器拒绝所有第三方 cookies，那么将不会被保存

## HTTP 响应首部字段

-   Access-Control-Allow-Origin

```
Access-Control-Allow-Origin: <origin> | *
```

origin 参数指定了允许访问该资源的外部域 URI。对于不需要携带身份认证的请求，服务器可以指定该字段为通配符，表示允许所有域的请求。

> 如果服务端指定了具体的域名而非“\*”，那么响应首部中的 Vary 字段的值必须包含 Origin。这将告诉客户端：服务器对不同的源站返回不同的内容。

-   Access-Control-Expose-Headers

在跨源访问时，XMLHttpRequest 对象的 getResponseHeader()方法只能拿到一些最基本的响应头，Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma，如果要访问其他头，则需要服务器设置本响应头。

Access-Control-Expose-Headers 头让服务器把允许浏览器访问的头放入白名单

```
Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header
```

这样浏览器就能够通过 getResponseHeader 访问 X-My-Custom-Header 和 X-Another-Custom-Header 响应头了。

-   Access-Control-Max-Age

指定了 preflight 请求的结果能够被缓存多久

```
Access-Control-Max-Age: <delta-seconds>
```

delta-second 指定预检请求的结果在多少秒内有效。

-   [Access-Control-Allow-Credentials](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials)

指定了当浏览器的 credentials 设置为 true 时是否允许浏览器读取 response 的内容。当用于预检请求时，它指定了实际请求中是否可以使用 credentials，简单请求不会发出预检请求，如果对此类请求的响应中不存在该字段，这个响应会被忽略掉，并且浏览器也不会将相应内容返回网页。

```
Access-Control-Allow-Credentials: true
```

-   Access-Control-Allow-Methods

用于预检请求的响应，表示实际请求所允许使用的 HTTP 方法。

```
Access-Control-Allow-Methods: <method>[, <method>]*
```

-   [Access-Control-Allow-Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Allow-Headers)

用于预检请求的响应，指明实际请求所允许携带的首部字段。

```
Access-Control-Allow-Headers: <field-name>[, <field-name>]*
```

## HTTP 请求首部字段

-   Origin

预检请求或者实际请求的源站点，不包含路径信息，只是服务器名词。

```
Origin: <origin>
```

> 有时候将该字段的值设置为空字符串是有用的，例如，当源站是一个 data URL 时。

> 注意，在所有访问控制请求（Access control request）中，Origin 首部字段总是被发送。

-   Access-Control-Request-Method

用于预检请求，告诉服务器实际请求所使用的 HTTP 方法。

```
Access-Control-Request-Method: <method>
```

-   Access-Control-Request-Headers

用于预检请求，告诉服务器实际请求所携带的首部字段。

```
Access-Control-Request-Headers: <field-name>[, <field-name>]*
```
