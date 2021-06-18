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

通过标记为 Secure 属性的 Cookie 应当被 HTTPS 协议加密后发送给服务器，因此可以预防中间人攻击，但是尽管如此，也不应在 Cookie 中存储敏感信息，毕竟 Cookie 是存储在本地的。

> 从 Chrome 52 和 Firefox 52 开始，不安全的站点（http:）无法使用 Cookie 的 Secure 标记。

-   HttpOnly

Document.cookie 无法获取 HttpOnly 标记的 Cookie；可以缓解 xss 攻击

### Cookie 的作用域

允许 Cookie 应该发送给哪些 URL

-   Domain

规定那些主机可以接受 Cookie，不指定默认为 Origin 不包含子域名。指定后一般包含子域名。

eg: Domain=mozilla.org 则可以包含在 developer.mozilla.org 中。

> 当前大多数浏览器遵循 RFC 6265，设置 Domain 时 不需要加前导点。浏览器不遵循该规范，则需要加前导点，例如：Domain=.mozilla.org

-   Path

规定了主机下那个路径可以接受 Cookie（该 URL 路径必须存在于请求 URL 中）。以字符 %x2F(/)作为路径分隔符号，子路径也会匹配。

eg: 设置，Path=/docs，则以下地址都会匹配

    /docs

    /docs/Web/

    /docs/Web/HTTP

### SameSite 属性

允许服务器要求某个 cookie 在跨站请求时不会被发送

-   None

    浏览器会在同站、跨站请求下继续发送 Cookie，不区分大小写。

-   Strict

    浏览器只在相同站点时发送 Cookie（在原有 Cookies 的限制条件上的加强，如上文 “Cookie 的作用域” 所述）

-   Lax

    类似 Strict，但是当用户从外部站点导航至 URL 时（通过链接）除外。在新版本浏览器为默认选项，Same-site cookies 将会为一些跨站子请求保留，如图片加载或者 frames 的调用，但只有当用户从外部站点导航到 URL 时才会发送。如 link 链接

### Cookie 前缀

服务器无法确定 Cookie 是否是安全来源设定的，甚至无法确定最初的设置来源。子域易受攻击的应用程序可以使用 Domain 属性设定 Cookie，从而可以在用户访问父域（或者其他子域）上的页面时，应用程序可能会信任用户 Cookie 发送的现有值。这可能允许用户在登陆后绕过 CSPF 保护或劫持会话。

> 在应用程序服务器上，Web 应用程序必须检查完整的 cookie 名称，包括前缀 —— 用户代理程序在从请求的 Cookie 标头中发送前缀之前，不会从 cookie 中剥离前缀。

-   \_\_Host-

    如果 Cookie 存在此前缀，并且它存在 Secure 属性标记，也是从安全来源发送的，不包括 Domain 属性，Path 属性也为/时，它才在 Set-Cookie 标头中接受（domain-locked）。

-   \_\_Secure-

    与 \_\_Host- 前缀相似，但是限制弱，如果存在此前缀，并且它存在 Secure 属性标记，是从安全来源发送的，它才在 Set-Cookie 标头中接受。
