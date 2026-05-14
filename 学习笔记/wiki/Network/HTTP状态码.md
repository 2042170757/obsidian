# HTTP 状态码

## 概述
HTTP 状态码是服务器返回的三位数字，指示请求的结果。分为 5 大类（1xx-5xx），由 RFC 9110 Section 15 定义。

## 核心内容

### 五大分类

| 类别 | 范围 | 含义 |
|------|------|------|
| **1xx** | 100-199 | 信息性：请求已收到，继续处理 |
| **2xx** | 200-299 | 成功：请求被成功接收、理解和接受 |
| **3xx** | 300-399 | 重定向：需要进一步操作 |
| **4xx** | 400-499 | 客户端错误：请求有误 |
| **5xx** | 500-599 | 服务器错误：服务器处理失败 |

### 常用状态码详解

#### 2xx 成功
| 码 | 名称 | 典型使用 |
|----|------|---------|
| 200 | OK | GET 成功 |
| 201 | Created | POST 创建成功 |
| 202 | Accepted | 异步任务已接受 |
| 204 | No Content | DELETE/PUT 成功，无返回内容 |

#### 3xx 重定向
| 码 | 名称 | 方法变化 |
|----|------|---------|
| 301 | Moved Permanently | 允许变 GET |
| 302 | Found | 允许变 GET（历史原因） |
| 303 | See Other | 强制变 GET |
| 304 | Not Modified | 使用缓存 |
| 307 | Temporary Redirect | **保持原方法** |
| 308 | Permanent Redirect | **保持原方法** |

#### 4xx 客户端错误
| 码 | 名称 | 说明 |
|----|------|------|
| 400 | Bad Request | 请求语法错误 |
| 401 | Unauthorized | 未认证（需登录） |
| 403 | Forbidden | 无权限（已认证但不允许） |
| 404 | Not Found | 资源不存在 |
| 405 | Method Not Allowed | 服务器认识该方法，但目标资源不支持 |
| 409 | Conflict | 资源冲突 |
| 422 | Unprocessable Content | 语法正确但语义有误 |
| 429 | Too Many Requests | 请求频率过高 |

#### 5xx 服务器错误
| 码 | 名称 | 说明 |
|----|------|------|
| 500 | Internal Server Error | 服务器内部错误 |
| 501 | Not Implemented | 服务器不认识该方法 |
| 502 | Bad Gateway | 网关/上游服务器错误 |
| 503 | Service Unavailable | 服务暂不可用 |
| 504 | Gateway Timeout | 网关超时 |

### 特殊状态码辨析

#### 401 vs 403

| 维度 | 401 Unauthorized | 403 Forbidden |
|------|-----------------|---------------|
| **语义** | 未认证（Unauthenticated） | 无权限（Unauthorized） |
| **客户端身份** | 服务器不知道你是谁 | 服务器知道你是谁 |
| **是否可重试** | 是，提供凭据后重试 | 否 |
| **响应头** | 必须包含 WWW-Authenticate | 不需要 |
| **类比** | "请出示证件" | "你不能进" |

虽然标准名为 "Unauthorized"，但 MDN 明确指出语义上实际是 "Unauthenticated"。

#### 405 vs 501

| 维度 | 405 Method Not Allowed | 501 Not Implemented |
|------|----------------------|---------------------|
| **语义** | 服务器**认识**该方法，但资源**不支持** | 服务器**不认识**该方法 |
| **响应头** | 必须包含 Allow 列表 | 不要求 |
| **类比** | "我知道 DELETE，但这个资源不支持" | "我不知道 PATCH 是什么" |

### 状态码排查流程

```
确定分类（第一个数字）
├── 2xx → 检查业务逻辑
├── 3xx → 检查重定向配置
├── 4xx → 检查客户端请求
│   ├── 400 → 参数格式、JSON 语法
│   ├── 401 → Token/Cookie 是否过期
│   ├── 403 → 用户权限、IP 白名单
│   ├── 404 → URL 路径是否正确
│   ├── 405 → 请求方法是否正确
│   ├── 422 → 字段校验失败
│   └── 429 → 等待后重试
└── 5xx → 检查服务器端
    ├── 500 → 查看服务器错误日志
    ├── 502 → 检查上游服务器
    ├── 503 → 服务器过载或维护
    └── 504 → 网络或超时配置
```

**实用技巧**：
- 405 响应会通过 Allow 头列出支持的方法
- 401 响应会通过 WWW-Authenticate 头指定认证方式
- 使用 `curl -v` 查看完整 HTTP 交互
- 查看 `X-Request-Id` 追踪请求

## 相关概念
- [[wiki/Network/HTTP基础]] - HTTP 协议基础
- [[wiki/Network/HTTP方法]] - HTTP 请求方法详解
- [[wiki/Network/CORS跨域]] - 跨域与预检请求

## 来源
- RFC 9110, Section 15 — https://rfc-editor.org/rfc/rfc9110#section-15
- IANA HTTP Status Code Registry — https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
- MDN: HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status
