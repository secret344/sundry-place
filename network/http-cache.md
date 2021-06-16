# http 缓存

重复使用已经获取的资源，可以显著提升网站个程序的性能。缓存减少了网络请求时间和流量，因此减少了显示该资源表示形式所需要的时间。

## 缓存类型

### 私有缓存

私有缓存只能用于单个用户，一般存在客户端。

### 共享缓存

共享缓存存储的响应能够被多个用户使用。一般为代理缓存，除此之外还有网关缓存、CDN、反向代理缓存和负载均衡器等部署在服务器上的缓存方式。

## 缓存目的

虽然 HTTP 缓存不是必须的，但是重复使用缓存的资源是必要的。缓存可以显著降低网络拥堵与延迟。

## 缓存控制

HTTP/1.1 定义了 **[Cache-Control](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)**头用来区分队缓存机制的支持情况。请求头和响应头都支持。

-   没有缓存
    > 每次由客户端发起的请求都会下载完整的响应内容。

```
    Cache-Control: no-store
```

-   缓存但重新验证
    > 此时，浏览器缓存会将此请求发送到服务器（会携带与本地缓存相关的验证字段）,服务端会确认请求中所描述的缓存资源是否过期，未过期返回 304 状态码，缓存使用本地缓存副本。

```
    Cache-Control: no-cache
```

-   私有，公共缓存
    -   public
        > 公共缓存，可以被中间人（CDN,代理服务器）缓存。
    -   private 私有缓存（默认）,响应是单个用户的，只会被浏览器私有缓存，中间人不能缓存响应。

```
    Cache-Control: private
    Cache-Control: public
```

-   过期
    > 过期机制中，最重要的是 "max-age=\<seconds>",表示最大缓存时间，相对于 Expires 来说，max-age 是距离请求发起的时间的**秒数**。针对应用中那些不会改变的文件，通常可以手动设置一定的时长以保证缓存有效，例如图片、css、js 等静态资源。

```
    Cache-Control: max-age=31536000
```

-   验证方式
    > must-revalidate 被使用时，已经过期的缓存资源不能被再次使用，必须先向服务器验证状态。

```
    Cache-Control: must-revalidate
```

-   Pragma
    > HTTP/1.0 标准定义的一个 header 属性，用来向后兼容只支持 HTTP/1.0 协议的缓存服务器，因为 1.1 标准没有明确定义这个属性，所以不能用来替代 Cache-Control

```
    Pragma=no-cache
```

与 Cache-Control: no-cache 效果一致。强制要求缓存服务器在返回缓存的版本之前将请求提交到源头服务器进行验证。

## 缓存验证

### 新鲜度

当资源被缓存时，由于本地空间有限，缓存会定期删除一些副本（缓存驱逐）。另一方面，服务器资源进行更新时，缓存对应资源也应当更新，但是服务器更新资源不可能通知每一个客户端进行缓存更新，所以必须约定一个过期时间，当过了过期时间，则这个缓存副本就是陈旧的。驱逐算法用来将陈旧的缓存副本替换为新鲜的。一个陈旧的缓存副本是不会被直接清楚或忽略的，当客户端发起请求时，缓存检索到一个对应已有的陈旧缓存副本，则缓存会附加一个 If-None-Match 头发送给目标服务器来检查资源副本此时的状态。服务器返回 304 则表示缓存状态是新鲜的，若服务器通过 If-None-Match 或者 If-Modified-Since 判断缓存过期，那么会返回最新的资源实体内容。

-   Expires
    > Expires 响应头包含日期/时间， 即在此时候之后，响应过期。如果在 Cache-Control 响应头设置了 "max-age" 或者 "s-max-age" 指令，那么 Expires 头会被忽略。
-   If-None-Match
    > 一个条件式请求首部
    ```
        If-None-Match: <etag_value>
        If-None-Match: <etag_value>, <etag_value>, …
        If-None-Match: *
    ```
    -   \<etag_value>
        > 请求实体
    -   \*
        > 任意资源，在资源上传时使用，通常时 PUT 方法，来检测是否拥有相同识别 ID 的资源已经被上传
-   If-Modified-Since

    -   服务器只在所请求的资源在给定的日期时间之后对内容进行过修改的情况下才会将资源返回,如果请求的资源从那时起未经修改，那么返回一个不带有消息主体的 304 响应，而在 Last-Modified 首部中会带有上次修改时间。
    -   当存在 If-None-Match 时候它会被忽略，除非服务器不支持 If-None-Match。

    -   条件式请求首部,只可以用在 GET 或 HEAD 请求中。

    ```
        If-Modified-Since: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT
    ```

-   If-Unmodified-Since
    > 只有当资源在指定的时间之后没有进行过修改的情况下，服务器才会返回请求的资源，或是接受 POST 或其他 non-safe 方法的请求。如果所请求的资源在指定的时间之后发生了修改，那么会返回 412 (Precondition Failed) 错误。

对于含有特定信息的头，回去计算缓存寿命。比如 Cache-Control: max-age=N，相应的缓存寿命就是 N。通常不含这个属性是回去查看 Expires 属性，通过比较 Expires 值与头里面的 Date(报文创建的日期和时间) 属性的值来判断是否缓存有效。如果 max-age 和 expires 属性都不存在，会查找头里面的 Last-Modified 信息，如果有，缓存的寿命就等于头里面 Date 的值减去 Last-Modified 的值除以 10（rfc2626）。

缓存失效时间计算公式如下：

```
    expirationTime = responseTime + freshnessLifetime - currentAge
    responseTime 表示浏览器接收到此响应的那个时间点。
```

### 校验

-   ETags 强校验
    -   资源的特定版本标识符，对于浏览器这样的 HTTP UA，不知道 ETag 代表什么，不能预测它的值是多少。如果资源请求的响应头里含有 ETag, 客户端可以在后续的请求的头中带上 If-None-Match 头来验证缓存。
    -   搭配 If-Match(条件请求) 来避免空中碰撞,与 If-Unmodified-Since 作用相似，但是 ETag 更加精确。
-   Last-Modified 弱校验
    -   说它弱是因为它只能精确到一秒。
    -   如果响应头里含有这个信息，客户端可以在后续的请求中带上 If-Modified-Since 来验证缓存。

## Vary 响应

-   HTTP 响应头决定了对于后续的请求头，如何判断是请求一个新的资源还是使用缓存的文件。
-   当缓存服务器收到一个请求，只有当前的请求和原始（缓存）的请求头跟缓存的响应头里的 Vary 都匹配，才能使用缓存的响应

eg:

1.  区分移动端还是桌面端,避免在不同的终端展示错误的布局。可以告诉搜索引擎没有引入[Cloaking](https://en.wikipedia.org/wiki/Cloaking)。

```
    Vary: User-Agent
```
