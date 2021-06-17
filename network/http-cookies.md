# http cookies

服务器发送到浏览器并保存在本地的一小块数据(一般 4kb),他会在浏览器下次向同一服务器发送请求时携带。

> 在 Web storage API 以及 IndexedDB 等本地存储功能未实现时，一度被用来保存数据，由于每次请求都会携带。会带来额外开销，所以浏览器都已允许开发者直接将数据存到本地。

## 主要用途

-   会话状态(购物车，登陆状态)
-   个性化设置(自定义设置，主题)
-   浏览器行为跟踪(分析用户行为)

## 创建 Cookie

当服务器收到 HTTP 请求时，可以在响应头部添加 Set-Cookie 选项，浏览器接受到响应会保存下 Cookie，之后每一次请求都会携带。另外，Cookie 的过期时间、域、路径、有效期、适用站点都可以根据需要来指定。

### Set-Cookie 响应头部和 Cookie 请求头部

服务器使用 Set-Cookie 响应头向用户（一般是浏览器）发送 Cookie 消息。

```
    Set-Cookie: <cookie name>=<cookie value>
    eg:
    HTTP/1.0 200 OK
    Content-type: text/html
    Set-Cookie: one_cookie=one
    Set-Cookie: two_cookie=two
    ...body
```

此后，客户端每次请求服务器都会携带之前保存的 Cookie 请求头部。

```
    GET /index_1.html HTTP/1.1
    Host: www.example.org
    Cookie: one_cookie=one; two_cookie=two
```

### Cookie 生命周期

定义 Cookie 生命周期可以通过 2 种方式：

-   会话期

会话时期不需要指定 Expires 或者 Max-Age，浏览器关闭之后他会自动删除。有些浏览器提供了会话恢复功能，这种情况下即使关闭了浏览器，会话期 Cookie 也会被保留下来，就好像浏览器从来没有关闭一样，这会导致 Cookie 的生命周期无限期延长。

-   持久性

持久性 Cookie 的生命周期取决于过期时间（Expires）或有效期（Max-Age）指定的一段时间。

1. Expires
2. Max-Age

### 限制访问 Cookie

保证 Cookie 在发送时不被其他脚本或者其他参与者（例如 xss,中间人攻击）访问。

-   Secure
-   HttpOnly
