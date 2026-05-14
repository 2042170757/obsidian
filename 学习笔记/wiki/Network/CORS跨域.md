# CORS 跨域

## 概述
CORS（Cross-Origin Resource Sharing）是浏览器的安全机制，基于同源策略，通过 HTTP 头控制跨域资源访问。本文涵盖同源策略、CORS 两种请求类型、配置方法和进阶场景选型。

## 核心内容

### 同源策略

同源策略是浏览器的核心安全机制。**同源**定义：协议 + 域名 + 端口三者完全相同。

| URL A | URL B | 是否同源 | 原因 |
|-------|-------|---------|------|
| `http://a.com` | `http://a.com/page` | 同源 | — |
| `http://a.com` | `https://a.com` | 跨域 | 协议不同 |
| `http://a.com` | `http://api.a.com` | 跨域 | 域名不同 |
| `http://a.com:80` | `http://a.com:8080` | 跨域 | 端口不同 |

### 为什么需要 CORS

同源策略防止恶意网站读取其他网站的敏感数据。例如：`evil.com` 的 JS 向 `bank.com/api/balance` 发请求时，浏览器会自动附带 bank.com 的 Cookie，如果没有同源策略，余额就会被窃取。

CORS 的设计思想：**服务器明确告知浏览器哪些跨域请求是安全的**。

**核心逻辑**：CORS 中请求**已经到达服务器**并被正常处理。是**浏览器**根据 CORS 响应头决定是否将响应交给 JavaScript。

### 两种请求类型

#### 简单请求

满足以下所有条件：方法仅限 GET/HEAD/POST；头部仅限 Accept、Accept-Language、Content-Language、Content-Type（且值仅限 `application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`）。

简单请求**不触发预检**，直接发送。浏览器在响应中检查 `Access-Control-Allow-Origin`。

#### 预检请求（Preflight）

不满足简单请求条件时（如 PUT/DELETE 方法、JSON Content-Type、自定义头），浏览器先发送 OPTIONS 预检请求：

```
1. 浏览器 → OPTIONS /api/users（预检）
   Access-Control-Request-Method: DELETE
   Access-Control-Request-Headers: Authorization

2. 服务器 → 204 No Content
   Access-Control-Allow-Origin: http://frontend.com
   Access-Control-Allow-Methods: GET, POST, PUT, DELETE
   Access-Control-Allow-Headers: Authorization, Content-Type
   Access-Control-Max-Age: 86400

3. 浏览器 → DELETE /api/users/123（实际请求）
```

### CORS Max-Age 浏览器差异（重要）

`Access-Control-Max-Age` 指定预检结果可缓存的最大秒数：

| 浏览器 | 最大缓存时间 |
|--------|------------|
| Firefox | **24 小时**（86400 秒） |
| Chromium < v76 | **10 分钟**（600 秒） |
| Chromium >= v76 | **2 小时**（7200 秒） |
| Safari/WebKit | **10 分钟**（600 秒） | WebKit 硬编码上限 |
| **默认值** | **5 秒**（未指定时） |

**建议**：设置 `Access-Control-Max-Age: 86400`，Chrome 会自动限制到 7200。

### 服务端配置

**核心 CORS 响应头**：

| 响应头 | 作用 |
|--------|------|
| `Access-Control-Allow-Origin` | 允许的来源 |
| `Access-Control-Allow-Methods` | 允许的方法 |
| `Access-Control-Allow-Headers` | 允许的请求头 |
| `Access-Control-Allow-Credentials` | 是否允许携带凭据 |
| `Access-Control-Max-Age` | 预检缓存时间 |
| `Access-Control-Expose-Headers` | 允许 JS 读取的响应头 |

### 常见配置误区

1. **`Allow-Origin: *` 与 `Credentials: true` 冲突**：当 `Allow-Credentials: true` 时，不能用通配符 `*`
2. **多域名支持**：不能直接写多个域名，需动态检查 Origin 请求头后回显
3. **忘记处理 OPTIONS**：需确保 OPTIONS 返回正确的 CORS 头和 204
4. **缺少 `Vary: Origin`**：动态返回不同 CORS 头时必须设置，否则 CDN 可能返回错误的头

### CORS 不能做的事

- CORS 不能阻止请求被发送（简单请求和表单提交不受限）
- CORS 只能阻止 JavaScript 读取响应
- CORS 不能替代 CSRF 防护

## 相关概念
- [[wiki/Network/HTTP基础]] - HTTP 协议基础
- [[wiki/Network/HTTP方法]] - 请求方法（简单请求方法限制相关）
- [[wiki/Network/HTTP状态码]] - 403、405 等状态码
- [[wiki/Network/实时通信技术]] - SSE 和 WebSocket 也涉及跨域

## 来源
- Fetch Standard（CORS 权威定义）— https://fetch.spec.whatwg.org/
- MDN: CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- MDN: Same-origin policy — https://developer.mozilla.org/en-US/docs/Glossary/Same-origin_policy
- MDN: Access-Control-Max-Age — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Max-Age
