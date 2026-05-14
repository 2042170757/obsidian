# HTTP 方法

## 概述
HTTP 方法定义了请求的目的和语义。RFC 9110 定义了 9 个标准方法，每个方法具有 Safe（安全）和 Idempotent（幂等）两个核心属性。

## 核心内容

### 方法属性定义

- **Safe（安全）**：方法不会修改服务器上的资源。GET、HEAD、OPTIONS、TRACE 是安全的
- **Idempotent（幂等）**：多次相同请求的效果与一次请求相同。GET、PUT、DELETE、HEAD、OPTIONS、TRACE 是幂等的

### 标准方法一览

| 方法 | 语义 | Safe | Idempotent | 有 Body |
|------|------|------|------------|---------|
| GET | 获取资源表示 | Yes | Yes | 不推荐 |
| HEAD | 获取元数据（无 Body） | Yes | Yes | No |
| POST | 提交数据/创建资源 | No | No | Yes |
| PUT | 替换完整资源 | No | Yes | Yes |
| DELETE | 删除资源 | No | Yes | Optional |
| PATCH | 部分更新资源 | No | No | Yes |
| OPTIONS | 获取通信选项 | Yes | Yes | Optional |
| TRACE | 消息环回测试 | Yes | Yes | No |
| CONNECT | 建立隧道 | No | No | No |

### 请求头 Header

请求头传递请求的元数据，格式为 `Key: Value`。核心请求头：

| 头字段 | 作用 |
|--------|------|
| Host | 目标主机名（HTTP/1.1 必需） |
| User-Agent | 客户端软件信息 |
| Accept | 可接受的响应格式 |
| Content-Type | 请求体数据格式 |
| Authorization | 认证凭据 |

**常见误区**：自定义头不再需要 `X-` 前缀（RFC 6648 已弃用）。HTTP/2 中头字段名必须小写传输。

### 请求体 Body

请求体携带业务数据，主要用于 POST、PUT、PATCH。常见 Content-Type：

| Content-Type | 适用场景 |
|-------------|---------|
| `application/json` | RESTful API（最常用） |
| `application/x-www-form-urlencoded` | HTML 表单 |
| `multipart/form-data` | 文件上传 |
| `text/plain` | 纯文本 |

### GET 方法

- **语义**：请求获取资源的表示
- **Safe + Idempotent + Cacheable**
- 参数通过 URL 查询字符串传递，长度有限制（2048-8192 字符，因浏览器而异）
- 不推荐携带 Body（RFC 未禁止但语义上不推荐，大多数服务器忽略）

**误区**：GET 不是"绝对不能有副作用"。Safe 是语义约定，服务器可记录日志等，但不应有用户可见的状态变化。

### POST 方法

- **语义**：让服务器根据资源自身语义处理请求中的数据
- **Not Safe + Not Idempotent**
- 参数放在请求体中，无大小限制
- 典型场景：创建资源、表单提交、文件上传、触发处理

Roy Fielding 本人明确表示 REST 论文中没有提到 CRUD，POST 的通用用途是 "this action isn't worth standardizing"。

### PUT 与 DELETE

**PUT**（替换完整资源）：
- Idempotent：多次相同 PUT 请求效果相同
- PUT 是替换，不是部分更新（部分更新用 PATCH）
- 如果资源不存在，可以创建新资源

**DELETE**（删除资源）：
- Idempotent：第一次成功，后续返回 404 但资源状态不变
- 服务器可实现为软删除

### PATCH 的幂等性争议（重要）

RFC 5789 原文明确规定：

> "PATCH is neither safe nor idempotent as defined by [RFC2616], Section 9.1."

但同时指出：

> "A PATCH request can be issued in such a way as to be idempotent"

**关键细节**：
- RFC 将 PATCH 定义为**默认非幂等**
- 但允许特定实现使 PATCH 成为幂等的
- 非幂等的例子：`PATCH {"qty": "+1"}`（增量操作），执行两次结果不同
- 幂等的例子：`PATCH {"name": "张三"}`（设置操作），执行多次结果相同
- IANA 注册表中 PATCH 标记为 `Idempotent: no`

**实践建议**：不要依赖 PATCH 的幂等性。如果需要幂等更新，使用 PUT 或添加 `If-Match` 条件头。

### RESTful 最佳实践

REST 是 Roy Fielding 在 2000 年博士论文中提出的**架构风格**，而非 API 设计规范。

业界惯例的 HTTP 方法与 CRUD 映射：

| CRUD 操作 | HTTP 方法 | 典型状态码 |
|-----------|----------|-----------|
| Create | POST | 201 Created |
| Read | GET | 200 OK |
| Update | PUT / PATCH | 200 / 204 |
| Delete | DELETE | 204 No Content |

**重要**：这是业界惯例，不是 REST 的硬性要求。Fielding 说："Search my dissertation and you won't find any mention of CRUD or POST."

## 相关概念
- [[wiki/Network/HTTP基础]] - HTTP 协议基础
- [[wiki/Network/HTTP状态码]] - 状态码分类与排查
- [[wiki/Network/CORS跨域]] - 跨域与预检请求

## 来源
- RFC 9110, Section 9: Methods — https://rfc-editor.org/rfc/rfc9110#section-9
- RFC 5789: PATCH Method — https://rfc-editor.org/rfc/rfc5789
- IANA HTTP Method Registry — https://www.iana.org/assignments/http-methods/
- MDN: HTTP request methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods
- Roy Fielding 博士论文 — https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
- Roy Fielding 博客（2009） — https://roy.gbiv.com/untangled/2009/03
