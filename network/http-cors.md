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
