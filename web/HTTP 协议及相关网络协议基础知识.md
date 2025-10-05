# **HTTP 协议及相关网络协议基础知识**

HTTP（超文本传输协议）是一种在 Web 上传输数据的规则。它规定了客户端（比如浏览器）和服务器之间如何互相发送请求和回应数据。下面从几个方面介绍 HTTP 协议的核心知识点：

------

## 1. HTTP 请求方法

HTTP 定义了多种请求方法，用来告诉服务器你希望它做什么。常见的方法及用途如下：

| 方法        | 说明                                                       | 示例用途                        |
| ----------- | ---------------------------------------------------------- | ------------------------------- |
| **GET**     | 仅用于获取数据，不改变服务器上的任何数据                   | 请求一个网页、图片              |
| **POST**    | 将数据提交给服务器，通常用于表单提交或创建新资源           | 用户登录、注册、提交评论        |
| **PUT**     | 通过请求体传送数据来替换目标资源的全部内容                 | 更新用户信息或上传文件          |
| **DELETE**  | 删除服务器上的指定资源                                     | 删除某个用户或删除一条记录      |
| **HEAD**    | 与 GET 类似，但只返回响应头部，不返回消息体                | 检查网页是否更新或是否存在      |
| **OPTIONS** | 查询服务器支持哪些请求方法以及其其他选项                   | 调试 API 时查看服务器支持的功能 |
| **TRACE**   | 回显客户端发送的请求，可用于诊断请求在网络中经过了哪些环节 | 调试网络请求路径                |
| **CONNECT** | 建立隧道连接，常用于 HTTPS 加密连接                        | 建立安全的连接，通常由代理使用  |

> **易懂说明**
>
> - **GET 请求**：就像你翻开一本书，只是看书里的内容，不做修改；
> - **POST 请求**：相当于你填写一份表单，提交给服务端处理；
> - **PUT/DELETE**：用于更新或删除数据，就好比修改或删除笔记；
> - **HEAD/OPTIONS/TRACE/CONNECT**：主要用于调试和诊断，是一些辅助工具。

------

## 2. HTTP 状态码

服务器在回应请求时会返回一个“状态码”，用来告诉客户端请求是否成功以及可能出现了什么问题。状态码分为五个类别：

- **1xx（信息类）**
   表示请求已接收，继续处理中。
   例如：100（Continue）
- **2xx（成功类）**
   请求成功，服务器成功返回请求的数据。
   例如：200（OK），201（Created），204（No Content）
- **3xx（重定向类）**
   需要客户端采取进一步的操作才能完成请求。
   例如：301（永久移动），302（临时移动），304（未修改）
- **4xx（客户端错误）**
   请求有错误，服务器无法理解请求。
   例如：400（错误请求），401（未授权），403（禁止访问），404（未找到）
- **5xx（服务器错误）**
   服务器在处理请求时发生错误。
   例如：500（服务器内部错误），502（网关错误），503（服务不可用），504（网关超时）

> **易懂说明**
>
> - 当你访问网页并看到 “404 页面不存在” 时，就意味着服务器找不到你请求的页面；
> - “200 OK” 则表示一切正常，数据成功返回。

------

## 3. HTTP 头部参数

HTTP 头部用于在请求和响应中传递额外信息，为数据的传输加上“说明书”。常见的请求头和响应头如下：

### 3.1 常见的 **请求头**

| 请求头                | 说明                                                 | 示例                                                    |
| --------------------- | ---------------------------------------------------- | ------------------------------------------------------- |
| **Host**              | 指定请求的服务器域名及端口。                         | `Host: www.example.com`                                 |
| **User-Agent**        | 描述客户端信息，如浏览器、操作系统等。               | `User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)` |
| **Accept**            | 指定客户端希望接收的内容类型。                       | `Accept: text/html, application/json`                   |
| **Accept-Language**   | 指定首选语言。                                       | `Accept-Language: zh-CN, en-US`                         |
| **Accept-Encoding**   | 声明支持的内容压缩编码格式（如 gzip）。              | `Accept-Encoding: gzip, deflate`                        |
| **Connection**        | 指定是否需要持久连接。                               | `Connection: keep-alive`                                |
| **Content-Type**      | 指定请求体的媒体类型。                               | `Content-Type: application/json`                        |
| **Content-Length**    | 请求体字节长度。                                     | `Content-Length: 348`                                   |
| **Authorization**     | 认证信息，用于身份验证。                             | `Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`     |
| **Cookie**            | 客户端存储的 Cookie 信息。                           | `Cookie: sessionId=abc123; theme=light`                 |
| **Referer**           | 当前请求的来源地址。                                 | `Referer: https://www.example.com/previous-page`        |
| **Origin**            | 请求的原始源，用于安全策略验证（如 CORS）。          | `Origin: https://www.example.com`                       |
| **If-Modified-Since** | 限定只有在资源自指定时间后修改才返回完整数据。       | `If-Modified-Since: Sat, 29 Oct 2022 19:43:31 GMT`      |
| **If-None-Match**     | 只有当资源 ETag 不匹配时返回数据，通常用于缓存验证。 | `If-None-Match: "737060cd8c284d8af7ad3082f209582d"`     |
| **Range**             | 请求资源的部分内容。                                 | `Range: bytes=500-999`                                  |

### 3.2 常见的 **响应头**

| 响应头                          | 说明                               | 示例                                                         |
| ------------------------------- | ---------------------------------- | ------------------------------------------------------------ |
| **Content-Type**                | 指定响应体的媒体类型及字符编码。   | `Content-Type: text/html; charset=UTF-8`                     |
| **Content-Length**              | 响应体数据的字节长度。             | `Content-Length: 1024`                                       |
| **Content-Encoding**            | 响应体的压缩编码格式。             | `Content-Encoding: gzip`                                     |
| **Content-Language**            | 响应体内容所使用的语言。           | `Content-Language: en`                                       |
| **Cache-Control**               | 缓存策略说明。                     | `Cache-Control: no-cache`                                    |
| **Expires**                     | 指定资源过期的时间。               | `Expires: Thu, 01 Dec 2022 16:00:00 GMT`                     |
| **ETag**                        | 资源的实体标识，常用于缓存验证。   | `ETag: "737060cd8c284d8af7ad3082f209582d"`                   |
| **Last-Modified**               | 资源最后修改的时间。               | `Last-Modified: Tue, 15 Nov 2022 12:45:26 GMT`               |
| **Location**                    | 重定向时指定资源的新地址。         | `Location: https://www.example.com/new-location`             |
| **Set-Cookie**                  | 服务器设置客户端 Cookie 的信息。   | `Set-Cookie: sessionId=abc123; Max-Age=3600; Path=/; HttpOnly` |
| **Server**                      | 服务器软件标识。                   | `Server: Apache/2.4.41 (Ubuntu)`                             |
| **WWW-Authenticate**            | 指示客户端需要进行的认证方式。     | `WWW-Authenticate: Basic realm="Access to the staging site"` |
| **Access-Control-Allow-Origin** | 允许跨域访问的源，支持 CORS 策略。 | `Access-Control-Allow-Origin: *`                             |
| **Retry-After**                 | 建议客户端在指定时间后重试请求。   | `Retry-After: 120`                                           |

> **易懂说明**
>
> - 请求头就像是你在点菜时告诉服务员你要什么、对口味的要求；
> - 响应头则是服务员告诉你菜里用了什么材料、保质期多久的说明。

------

## 4. HTTP 请求与响应示例

### 4.1 HTTP 请求示例

假设你用浏览器请求首页，完整的请求可能如下：

```
GET /index.html HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN, en-US
Accept-Encoding: gzip, deflate
Connection: keep-alive
Cookie: sessionId=abc123
Referer: https://www.google.com/
```

**说明：**

- **首行**：`GET /index.html HTTP/1.1` 指定请求方法、目标资源（index.html）与协议版本。
- **Host**：指明目标服务器的域名。
- **User-Agent**：告知服务器使用的客户端信息。
- **Accept 系列**：说明浏览器支持和期望接收的内容类型和语言。
- **Connection**：保持持久连接以便后续请求。
- **Cookie** 和 **Referer**：分别携带之前保存的 Cookie 信息和页面来源。

### 4.2 HTTP 响应示例

服务器收到请求后返回的响应可能如下：

```
HTTP/1.1 200 OK
Date: Tue, 15 Apr 2025 12:00:00 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Type: text/html; charset=UTF-8
Content-Length: 1024
Connection: keep-alive
Set-Cookie: sessionId=abc123; Max-Age=3600; Path=/; HttpOnly
Cache-Control: no-cache
```

**说明：**

- **状态行**：`HTTP/1.1 200 OK` 表示请求成功，返回所需数据。
- **Date** 与 **Server**：标明响应发送时间和服务器软件信息。
- **Content-Type** 和 **Content-Length**：定义返回内容的类型、编码及长度。
- **Set-Cookie**：设置了一个新的 Cookie，便于后续维持会话。
- **Cache-Control**：告知客户端不要缓存此页面数据。

------

## 5. HTTP 无状态与身份认证技术

### 5.1 无状态的特性

- **无状态（Stateless）**
   HTTP 本身不保存客户端的上一次请求信息。每个请求都是独立的，就好比每次点菜时服务员都不会记得你上一次点了什么。如果要记住用户状态，就需要借助其他技术。

### 5.2 身份认证技术

常见的方式有：

|       认证方式       | 是否无状态 |                           工作原理                           |                          优缺点说明                          |
| :------------------: | :--------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| **Cookie + Session** |   有状态   | 用户登录后服务器生成 Session，将唯一标识通过 Cookie 返回。 服务器存储用户状态 | 优点：实现简单、兼容性好；缺点：服务器存储信息，扩展时需额外考虑 |
| **Token（如 JWT）**  |   无状态   | 用户登录后服务器生成包含用户信息的 Token（例如 JWT），客户端存储并在请求中携带 | 优点：无需存储状态，适合分布式和微服务架构；缺点：Token 被窃取风险 |
| **HTTP Basic 认证**  |   无状态   |    客户端每次请求时在请求头中附带Base64编码的用户名和密码    | 优点：简单易用，浏览器内置支持；缺点：安全性低，必须与 HTTPS 配合使用 |
| **HTTP Digest 认证** |   无状态   |  类似 Basic 认证，但通过哈希算法发送凭据，安全性比 Basic 高  | 优点：比 Basic 认证更安全；缺点：实现较复杂，浏览器支持不统一 |

> **易懂说明：**
>
> - Cookie+Session 就像是饭店的会员卡系统，服务器记住你的信息；
> - Token 认证则相当于发给你一个数字钥匙，你每次带着钥匙进入，服务器无需记住你是谁；
> - Basic 和 Digest 则是将用户名和密码直接或经过加密后随请求发送，一般用于简单内部测试。

------

## 6. 总结

- **HTTP 请求方法**：区分数据的获取、提交、更新和删除，让不同操作各司其职。
- **HTTP 状态码**：用数字告诉你请求是否成功，出错时还能告诉你是什么问题。
- **HTTP 头部**：用来传递附加说明和配置数据（比如浏览器类型、缓存、认证等）。
- **请求与响应报文**：每次交互都有固定的格式，便于解析和维护。
- **身份认证和状态管理**：尽管 HTTP 协议本身是无状态的，但可通过 Cookie/Session 或 Token 等机制管理用户状态，确保安全通信。