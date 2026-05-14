# HTTP 基础

## 概述
HTTP 是 Web 的基础协议，定义了客户端与服务器之间的通信规则。本文涵盖 HTTP 协议定义、工作流程、请求组成和 URL 结构。

## 核心内容

### 什么是 HTTP

HTTP（HyperText Transfer Protocol）是应用层协议，采用客户端-服务器模型和请求-响应模式。其核心特性是**无状态**：每个请求独立，服务器不保留之前的请求信息。

- HTTP 默认端口：**80**
- HTTPS 默认端口：**443**（TLS/SSL 加密）

无状态意味着服务器不记住客户端的之前请求。如果需要状态管理，需借助 Cookie、Session、Token 等机制。无状态设计使得服务器可以并行处理请求，提高可扩展性。

> "All REST interactions are stateless. That is, each request contains all of the information necessary for a connector to understand the request, independent of any requests that may have preceded it." — RFC 9110

**常见误区**：HTTP 不是连接协议，而是应用层协议，建立在 TCP 之上（HTTP/3 基于 QUIC/UDP）。

### HTTP 的工作过程

一次完整的 HTTP 通信：

1. **DNS 域名解析**：浏览器缓存 → 系统缓存 → 路由器缓存 → DNS 服务器
2. **TCP 三次握手**：SYN → SYN+ACK → ACK，建立可靠连接
3. **请求发送**：客户端发送 HTTP 请求报文
4. **响应接收**：服务器返回 HTTP 响应报文
5. **连接关闭或复用**：
   - HTTP/1.0：默认短连接
   - HTTP/1.1：默认长连接（Connection: keep-alive）
   - HTTP/2：多路复用
   - HTTP/3：基于 QUIC（UDP）

### HTTP 请求核心组成

| 组成部分 | 作用 | 说明 |
|---------|------|------|
| **URL** | 标识资源位置 | scheme://host/path?query#fragment |
| **Method** | 指示请求目的 | GET、POST、PUT 等 |
| **Header** | 传递元数据 | Key: Value 格式 |
| **Body** | 携带业务数据 | POST/PUT/PATCH 使用 |

### URL 完整结构（RFC 3986）

```
https://www.example.com:443/api/v1/users?page=1&size=10#profile
  ^       ^                ^   ^            ^              ^
协议    域名             端口 路径       查询参数        锚点
```

- **协议**：`https` 通过 TLS 加密，`http` 明文传输
- **域名**：不区分大小写。端口可省略（HTTP 默认 80，HTTPS 默认 443）
- **路径**：区分大小写
- **查询参数**：`?` 开头，`&` 分隔。特殊字符需 URL 编码
- **锚点**：仅客户端使用，**不发送到服务器**

**常见误区**：URL 和 URI 不同。URL 是 URI 的子集，URI 可以是 URL（定位资源）或 URN（命名资源）。

## 相关概念
- [[wiki/Network/HTTP方法]] - HTTP 请求方法详解
- [[wiki/Network/HTTP状态码]] - HTTP 状态码分类与排查
- [[wiki/Network/实时通信技术]] - SSE、WebSocket 等实时技术
- [[wiki/Network/CORS跨域]] - 跨域资源共享机制

## 来源
- RFC 9110: HTTP Semantics — https://rfc-editor.org/rfc/rfc9110
- RFC 3986: URI Generic Syntax — https://rfc-editor.org/rfc/rfc3986
- RFC 9293: TCP — https://rfc-editor.org/rfc/rfc9293
- MDN: HTTP overview — https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview
