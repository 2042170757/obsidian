---
title: 别跟 API 玩猜心：RESTful 的契约、JSON 的情书与三巨头统一“黑话”
date: 2026-04-28
tags:
  - API
  - REST
  - JSON
  - LLM
---

# 别跟 API 玩猜心：RESTful 的契约、JSON 的情书与三巨头统一“黑话”

## 为什么学这门课？

你写了一个 Python 脚本调用大模型，换了个厂商，代码崩了。你对接了一个第三方 API，文档翻了三遍还不知道该怎么传参数。你写了个后端接口，被前端同事吐槽“这玩意儿也叫 RESTful？”

这些问题都有一个共同的根：**API 的设计哲学、数据交换格式和调用规范**。

这篇笔记不讲枯燥的 RFC 原文，而是从实际痛点出发，聊清楚三件事：
1. RESTful API——前端和后端之间的“合同”该怎么写
2. JSON——数据交换的“通用语言”该怎么用
3. 大模型 API 格式——OpenAI、Anthropic、Gemini 的“黑话”能不能统一

---

## 什么是 API？

### 定义

**API（Application Programming Interface，应用程序编程接口）**，本质是一套预定义的规则，让不同的软件系统能够相互通信。

类比：餐厅里，**菜单**就是 API。你（客户端）看菜单点菜，厨房（服务器）按菜单做菜。你不需要知道厨房里是怎么炒的，厨房也不需要知道你是谁——只要按菜单上的规则来就行。

### 核心价值

- **解耦**：调用者不需要知道内部实现
- **复用**：一套接口能被多个客户端使用
- **标准化**：统一的通信协议和数据格式

### 常见类型

- **Web API**（RESTful、GraphQL、gRPC）——通过 HTTP 通信，最主流
- 库/框架 API——代码层面的函数调用接口
- 操作系统 API——如 Windows API、POSIX
- 硬件 API——如 GPU 驱动接口

本文聚焦 **Web API**，尤其是当前 AI 时代绕不开的那几种。

---

## REST 与 RESTful

### 是什么？有什么区别？

**REST（Representational State Transfer，表征状态转移）** 是 Roy Fielding 在 2000 年的博士论文中提出的一种**架构风格**（Architectural Style）。它是一套设计理念和约束条件，不是一个具体的实现或标准。

**RESTful** 是**遵循 REST 架构风格的具体 API 实现**的形容词。如果一个 API 严格遵守了 REST 的六大约束，它就是 RESTful 的。

> **关键区分**：
> - REST = 论文里的理论
> - RESTful = 实践中的实现（而且大多数“RESTful API”并没有严格遵守全部约束）

### 为什么选择 REST？

相对于早期的 SOAP（复杂、重量级）和纯粹的 RPC（混乱、无规范），REST 提供了：

- **简单**：基于 HTTP 协议，人人熟悉的 GET/POST/PUT/DELETE
- **无状态**：每个请求自包含，便于水平扩展
- **可缓存**：利用 HTTP 缓存机制，提升性能
- **松耦合**：客户端和服务端可以独立开发和部署

### REST 不是唯一的 API 架构

需要明确：REST 是主流，但不是唯一。了解其他架构有助于在正确场景做出正确选择。

| 架构          | 最佳场景                         | 劣势                                       |
| ----------- | ---------------------------- | ---------------------------------------- |
| **REST**    | 公网 API、简单 CRUD、需要 HTTP 缓存的场景 | 数据过获取（over-fetching）/欠获取（under-fetching） |
| **GraphQL** | 客户端需要灵活的数据形状、移动端/SPA         | 缓存困难、查询复杂度攻击风险、学习曲线较高                    |
| **gRPC**    | 内部微服务通信、高性能双向流、IoT           | 浏览器支持有限、人类不可读（Protobuf 二进制）              |

**现代最佳实践往往是混合架构**：内部微服务用 gRPC，BFF 层用 GraphQL，公网 API 用 REST。

---

## 核心原则 1：一切皆为资源

### URI 是资源的唯一标识

在 REST 中，**资源（Resource）** 是所有可以被访问和操作的数据实体，而 **URI（Uniform Resource Identifier）** 是资源的“身份证”和“地址”。

### 传统 RPC 风格 vs RESTful 风格

```text
RPC 风格（动作优先）：
  GET /getUser?id=123
  POST /createUser
  POST /deleteUser

RESTful 风格（资源优先）：
  GET    /users/123
  POST   /users
  DELETE /users/123
```

RESTful 以资源为中心，用 HTTP 方法来表达操作意图，而不是把动作塞进 URL。

---

## 核心原则 2：无状态通信

### 无状态：请求自包含

所谓“无状态”，是指**服务端不存储客户端的会话状态**。每个请求都包含处理该请求所需的全部信息。

**无状态的优势**：
- 水平扩展：任何服务器都可以处理任何请求，无需共享 session 存储
- 可靠性：服务器重启不影响客户端
- 可见性：请求可以被中介（代理、网关）检查和缓存

### 但无状态不是银弹

> **重要提醒**：以下内容比大多数 REST 教程说得更深入，请认真阅读。

**购物车场景**：电商购物车天生有状态。把购物车数据塞进 JWT 假装“无状态”只是一种技术游戏——一旦需要实时封禁账号、触发风控、限制并发登录等业务逻辑，无状态方案就会遇到严重障碍。

**JWT 算不算有状态？**
- 纯技术层面：JWT 把状态声明存在客户端，服务器不存 session，**算无状态**
- 业务层面：一旦引入 Redis 黑名单来吊销 token，本质上就是**重新发明了 session**——更复杂，但没有解决根本问题

> **结论**：无状态是工具箱中的一件利器，适合水平扩展场景。但遇到需要实时吊销、事务一致性、复杂状态机的业务时，有状态的 session 方案反而更务实。

---

## 核心原则 3：统一接口

### 用 HTTP 方法精准表达操作语义

RESTful API 通过标准的 HTTP 方法来表达对资源的操作：

| HTTP 方法 | 语义 | 安全性 | 幂等性 |
|-----------|------|--------|--------|
| **GET** | 获取资源 | 是 | 是 |
| **POST** | 创建资源 | 否 | 否 |
| **PUT** | 全量替换资源 | 否 | 是 |
| **PATCH** | 部分更新资源 | 否 | 通常幂等（取决于语义） |
| **DELETE** | 删除资源 | 否 | 是 |

> **特别注意**：很多教程只讲 GET/POST/PUT/DELETE，漏掉了 **PATCH**（RFC 5789）。PUT 是**全量替换**——不传的字段会被重置或删除；PATCH 是**部分更新**——只修改传了的字段。两者语义完全不同，混用会导致隐蔽的 bug。

**幂等性**：多次执行产生相同结果。例如 `PUT /users/123` 即使调用 10 次，只要 body 相同，最终状态一致。

---

## 核心原则 4：可缓存性

### 让请求更快，减少服务器压力

- 明确标注缓存策略（通过 `Cache-Control`、`ETag`、`Last-Modified` 等 HTTP 头）
- 客户端可以复用缓存的响应结果
- 减少不必要的请求，提升性能，降低成本

**最佳实践**：GET 请求的响应应该默认可缓存（除非有特殊原因不缓存）。

---

## 核心原则 5：客户端-服务器分离

### 独立开发，独立部署

- **客户端**：关心用户界面和数据展示
- **服务器**：关心数据存储和业务逻辑

两者通过 API 契约进行通信，互不干扰。前端可以换成 React 或 Vue，后端可以换成 Python 或 Go，只要 API 不变，双方都可以独立演进。

---

## 核心原则 6：分层系统

### 隐藏后端的复杂结构

客户端不需要知道请求背后有多少层：

```text
客户端 -> 负载均衡 -> 缓存层 -> API 网关 -> 微服务 -> 数据库
```

每一层只和相邻层交互，客户端无感知。这允许灵活插入中间层（如认证、限流、日志），而且所有层次仍然通过统一的 HTTP 接口交互。

---

## RESTful 背后的残酷真相：什么是“真 REST”？

> 这个部分大多数教程不会告诉你，但它很重要。

Roy Fielding 在 2008 年的一篇博客中愤怒地写道：“我看到无数人把任何基于 HTTP 的接口称为 REST API。那是 RPC。”

真正的 REST 架构有一个核心约束叫 **HATEOAS（Hypermedia as the Engine of Application State，超媒体作为应用状态引擎）**。简单说：**服务端返回的响应中应该包含下一步可以执行的操作链接**，客户端不应该硬编码任何 URL。

```json
// 真正的 REST（HATEOAS 风格）：
GET /users/123
{
  "id": 123,
  "name": "张三",
  "links": [
    { "rel": "self", "href": "/users/123" },
    { "rel": "articles", "href": "/users/123/articles" },
    { "rel": "edit", "href": "/users/123", "method": "PUT" }
  ]
}

// 工业界实际的做法（“其实只是 JSON over HTTP”）：
GET /users/123
{
  "id": 123,
  "name": "张三"
}
```

**为什么几乎没有 API 实现 HATEOAS？**
- JSON 没有 `<a>` 标签和 `<form>` 那样的原生超媒体能力——HTML 才有
- 真实客户端（SPA、移动 App）都是硬编码 URL，不需要运行时发现
- 工具链严重缺失：几乎没有 HTTP 客户端原生支持 HATEOAS

> **坦诚的事实**：你看到的 99% 的“RESTful API”，严格来说只是 **JSON over HTTP with RPC-style routing**。但工业界和学术界各走各路，这已经是共识——Fielding 的理想很好，但现实需要实用主义。

---

## RESTful 最佳实践

这部分讨论的是**实际工程中被广泛接受**的规范，而非学术层面的纯 REST。

### URI 命名：名词复数，拒绝动词

```text
推荐：
  GET    /users          -> 获取用户列表
  GET    /users/{id}     -> 获取单个用户
  POST   /users          -> 创建用户
  PUT    /users/{id}     -> 全量更新用户
  PATCH  /users/{id}     -> 部分更新用户
  DELETE /users/{id}     -> 删除用户

不推荐：
  GET /getUserList
  POST /createUser
  POST /deleteUser?id=123
```

**关于单数 vs 复数**：业界主流（GitHub、Stripe、Google、Postman 2024 报告）推荐复数。少数派认为单数（`/user/{id}`）在语义上更准确。**一致性比选择哪一方更重要。**

### 状态码：善用标准 HTTP 语义

常见误用对照表：

| 场景 | 正确状态码 | 常见误用 |
|------|-----------|----------|
| 获取成功 | `200 OK` | — |
| 创建成功 | `201 Created` | 返回 `200 OK` |
| 删除成功 | `204 No Content` | 返回带 body 的 `200 OK` |
| 请求格式错误 | `400 Bad Request` | 返回 `500` |
| 验证规则不通过 | `422 Unprocessable Entity` | 返回 `400` |
| 未认证 | `401 Unauthorized` | 混用 `403` |
| 无权限 | `403 Forbidden` | 混用 `401` |
| 资源不存在 | `404 Not Found` | 返回 `200` 带空 body |
| 频率限制 | `429 Too Many Requests` | 不返回，默默拒绝 |

**最严重的反模式**：所有请求都返回 `200 OK`，然后在 body 里塞状态码。这会破坏 HTTP 缓存机制、监控系统和客户端自动错误处理。

### 版本控制

推荐在 URI 中明确标注版本：

```text
GET /v1/users
GET /v2/users
```

### 分页与过滤

通过查询参数实现：

```text
GET /articles?page=2&limit=20&author=zhangsan&sort=created_at:desc
```

---

## RESTful 实操：设计博客 API

### 资源定义

- **User**（用户）
- **Article**（文章）
- **Comment**（评论）

### 标准接口

```text
# 用户
GET    /users              -> 用户列表
GET    /users/{id}         -> 单个用户
POST   /users              -> 创建用户
PUT    /users/{id}         -> 全量更新用户
PATCH  /users/{id}         -> 部分更新用户
DELETE /users/{id}         -> 删除用户

# 文章
GET    /articles           -> 文章列表
GET    /articles/{id}      -> 单篇文章
POST   /articles           -> 创建文章
PUT    /articles/{id}      -> 全量更新
PATCH  /articles/{id}      -> 部分更新
DELETE /articles/{id}      -> 删除文章

# 嵌套资源（用户下的文章）
GET    /users/{uid}/articles        -> 用户的文章列表
GET    /articles/{aid}/comments     -> 文章的评论列表
POST   /articles/{aid}/comments     -> 添加评论
```

### 进阶思考：如何获取“用户 789”的所有文章？

两种方案：

| 方案 | URI | 说明 |
|------|-----|------|
| 嵌套资源 | `GET /users/789/articles` | 语义清晰，固定路径 |
| 查询参数 | `GET /articles?author=789` | 更灵活，适合组合过滤 |

两者没有绝对优劣，按实际场景选择——**需要经常按用户查询就用嵌套，需要组合多种过滤就用查询参数。**

---

## 第一章节总结

### 核心要点

1. **REST** 是一套架构风格理论，**RESTful** 是遵循该理论的实践（但不一定完全遵守）
2. **六大约束**：资源标识、无状态通信、统一接口、可缓存性、客户端-服务器分离、分层系统
3. 无状态并非万能——购物车、实时吊销等场景有状态方案反而更务实
4. **PUT 和 PATCH 要区分**（全量替换 vs 部分更新）
5. **HATEOAS** 学术上很重要，工业界基本没实现——要清楚这个割裂
6. REST 不是唯一选择——GraphQL 和 gRPC 在特定场景下更优
7. 一致性比追求“完美规范”更重要

### 互动思考

问你一个问题：下面这个接口，是 RESTful 吗？

```text
POST /updateUserStatus
Body: { "id": 123, "status": "active" }
```

> **答案**：不是。URL 用了动词（`updateUserStatus`），正确姿势是：`PATCH /users/123 { "status": "active" }`。

---

# 走进 JSON：数据交换的通用语言

## 为什么 JSON 成了行业标准？

### JSON 的定义

**JSON（JavaScript Object Notation，JavaScript 对象表示法）** 是一种轻量级的数据交换格式，本质是**字符串**。

### 对比 XML

| 对比维度 | JSON | XML |
|----------|------|-----|
| 语法简洁度 | 极简 | 冗余（闭合标签） |
| 解析速度 | 快 | 慢 |
| 带宽占用 | 小 | 大（标签重复） |
| 可读性 | 良好 | 一般 |

### 跨语言支持

几乎所有主流编程语言都内置了 JSON 解析和序列化支持：

- Python：`json` 标准库
- JavaScript：`JSON.parse()` / `JSON.stringify()`
- Java：`Jackson` / `Gson`
- Go：`encoding/json`
- Rust：`serde_json`

### JSON 是“最佳”数据格式吗？

> **更准确的说法**：JSON 是**数据交换的事实标准**，但并非在所有场景下都是“最佳”。

| 格式 | 优势 | 劣势 |
|------|------|------|
| **JSON** | 人类可读、普及率高、浏览器原生 | 慢、胖、无类型 |
| **Protobuf** | 快 5-7x、小 2-3x、强类型合约 | 不可读、需 .proto 文件 |
| **MessagePack** | 比 JSON 快 2-4x、小 1.7x、无需 schema | 生态不如 JSON |

对于**公网 API 和需要人类调试的场景**，JSON 是正确选择。对于**内部高性能微服务通信**，Protobuf 或 MessagePack 更优。

---

## JSON 的基础语法

### 核心结构

```json
{
  "name": "张三",
  "age": 25,
  "hobbies": ["coding", "reading"],
  "address": {
    "city": "北京",
    "zip": "100000"
  }
}
```

- **对象（Object）**：花括号 `{}`，键值对集合
- **数组（Array）**：方括号 `[]`，有序列表
- **嵌套（Nesting）**：对象和数组可以任意嵌套

---

## JSON 支持的数据类型

| 类型 | 示例 | 注意 |
|------|------|------|
| 字符串 | `"hello"` | **必须用双引号** |
| 数字 | `42`, `3.14`, `-1`, `1e10` | 不区分整数和浮点数，所有解析器用 IEEE 754 双精度处理 |
| 布尔值 | `true`, `false` | 小写 |
| 空值 | `null` | 不是 `undefined` |
| 对象 | `{}` | 键必须用双引号 |
| 数组 | `[]` | 元素可以是任意类型 |

### 警惕：JSON 数字精度问题

这是一个**非常重要但经常被忽略的坑**：

JSON 不区分整数和浮点数。所有数字都以 IEEE 754 双精度浮点数解析。这意味着：

- **安全整数范围**：`-(2^53 - 1)` 到 `2^53 - 1`（约 +-9 千万亿）
- 超出此范围的整数会被**静默截断**且不报错：

```javascript
JSON.parse("9007199254740993")
// 返回: 9007199254740992  <- 数据丢失了！没有报错！
```

**真实影响场景**：
- 数据库 64 位 ID（如 MySQL BIGINT）
- 区块链交易金额
- 金融系统的精确计算
- Twitter API 的 tweet ID

**最著名的 Workaround**：Twitter 的 `id` + `id_str` 双字段模式——一个保持数字兼容，一个存为字符串以保证精度。

---

## JSON 的常见语法坑点

### 禁止单引号

```json
// 错误
{ 'name': '张三' }

// 正确
{ "name": "张三" }
```

### 禁止尾随逗号

这是一段有趣的历史：ES5（2009年）为了修复 IE 和 其他浏览器对数组长度计算不一致的 bug，标准化了尾随逗号。但 JSON 规范始终禁止它。

```json
// 错误
{ "name": "张三", "age": 25, }

// 正确
{ "name": "张三", "age": 25 }
```

很多开发者混淆了 **JS 对象字面量**（允许尾随逗号）和 **JSON**（不允许尾随逗号）。JSON5 项目试图弥合这个鸿沟，但从未被官方 JSON 规范采纳。

### 禁止添加注释

**Douglas Crockford（JSON 的创造者）** 在 2012 年明确解释过原因：“我移除注释是因为我看到人们用注释来存放解析指令，这会破坏互操作性。”

他的逻辑：JSON 是**数据交换格式**不是**配置文件格式**。注释是元数据，不是数据。

**争议**：批评者认为这是“一个人犯错，所有人受罚”。现实中，VSCode 的 `settings.json`、TypeScript 的 `tsconfig.json` 等配置文件都默默支持了注释，形成了 JSONC（JSON with Comments）的非标准生态。

> **实际建议**：如果一定要加注释，用 `"_comment": "这里是注释"` 这种约定。

### 禁止 undefined 类型

```json
// undefined 不是有效的 JSON 值
{ "name": undefined }  // 错误
{ "name": null }       // 正确
```

---

## JSON 的读取与解析

### 核心概念

- **解析（Parsing）**：把 JSON 字符串 -> 程序中的数据结构
- **序列化（Serialization）**：把程序中的数据结构 -> JSON 字符串

### Python 标准实现

```python
import json

# 序列化：Python 对象 -> JSON 字符串
data = {"name": "张三", "age": 25}
json_str = json.dumps(data, ensure_ascii=False)  # ensure_ascii=False 保留中文
# 输出: {"name": "张三", "age": 25}

# 解析：JSON 字符串 -> Python 对象
parsed = json.loads(json_str)
print(parsed["name"])  # 张三
```

---

## JSON 在 API 中的作用

### 请求与响应的标准载体

```text
请求 (Client -> Server):
  POST /users
  Content-Type: application/json
  Body: {"name": "张三", "email": "zhangsan@example.com"}

响应 (Server -> Client):
  HTTP/1.1 201 Created
  Content-Type: application/json
  Body: {"id": 456, "name": "张三", "email": "zhangsan@example.com", "created_at": "2026-04-28T12:00:00Z"}
```

### JSON Schema：给 JSON 加上“身份证”

随着 API 越来越复杂，纯 JSON 的问题暴露出来：**没有类型约束，没有文档**。

**JSON Schema** 就是给 JSON 数据定义“模板”：

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "name": { "type": "string", "minLength": 1 },
    "age":  { "type": "integer", "minimum": 0 }
  },
  "required": ["name", "age"]
}
```

**生态现状**（2024-2025）：
- 2024 年举办了首届 JSON Schema 大会，赞助商从 4 家增长到 15 家
- **OpenAPI 3.1** 已完全集成 JSON Schema（弃用自己的 `nullable` 等扩展属性）
- **OpenAI Structured Outputs** 和 **Anthropic MCP** 协议都重度依赖 JSON Schema
- 但 JSON Schema 也有批评者，认为它对代码生成不友好——微软推出了 TypeSpec，AWS 推出了 Smithy，作为替代

---

## 第二章节总结

### 语法核心

- 两种结构：对象（`{}`）和数组（`[]`）
- 六种类型：字符串、数字、布尔值、null、对象、数组
- 三大禁令：**禁止单引号、禁止尾随逗号、禁止注释**

### 解析与序列化

- JSON 本质是字符串
- **解析** = 字符串 -> 数据结构
- **序列化** = 数据结构 -> 字符串

### 精度警告

- 超过 `2^53 - 1` 的整数会静默丢失精度——需用字符串传递

### 互动找 bug

```json
{
  'name': '张三',
  'hobbies': ['coding'], // 注释
}
```

> 三个 bug：
> 1. 单引号（应用双引号）
> 2. 尾随逗号（数组后多了逗号）
> 3. 注释（JSON 不支持 `//`）

---

# 大模型 API 的格式痛点

## 为什么这是个问题？

截至 2026 年，大模型市场三足鼎立：

| 厂商 | 代表模型 | 技术路线 |
|------|----------|----------|
| **OpenAI** | GPT-5 / GPT-5-mini | 原生格式 + 市场主导 |
| **Anthropic** | Claude Opus 4.5 / Sonnet 4.5 | 独立格式 + 后发兼容 |
| **Google** | Gemini 2.5 Pro / 2.0 Flash | 原生格式 + 兼容模式 |

**问题**：三家 API 格式不统一，切换模型需要改写大量适配代码。

---

## OpenAI 兼容格式：事实标准？

### 它是“行业标准”吗？

**事实上的市场领导者（de facto standard）**——但不是正式的行业标准（de jure standard）。

- **支持的证据**：Anthropic（2025 年 5 月）和 Google（2025 年 4 月）都推出了 OpenAI 兼容端点
- **反对的证据**：有分析（如 Latent Space 的《Industry Standard or Strategic Trap?》）认为这可能变成“温水煮青蛙”——越习惯 OpenAI 格式，切换成本越高

### 请求结构

```python
from openai import OpenAI

client = OpenAI(api_key="sk-xxx")

response = client.chat.completions.create(
    model="gpt-5",          # 模型名称
    messages=[               # 消息列表
        {"role": "system", "content": "你是AI助手"},
        {"role": "user", "content": "你好"}
    ],
    temperature=0.7,         # 控制随机性
    max_tokens=1000          # 最大输出长度
)
```

### 响应结构

```python
# response 对象的主要字段
response.id                # 请求唯一标识
response.model             # 使用的模型
response.usage             # 计费统计
response.choices[0].message.content   # 模型回复文本
response.choices[0].finish_reason     # 结束原因（stop/length/tool_calls）
```

### 统一返回格式的优势

```python
# 无论调用哪个模型，response 的结构都是一致的
print(response.choices[0].message.content)
```

---

## Anthropic 独立格式

### 请求结构

```python
from anthropic import Anthropic

client = Anthropic(api_key="sk-ant-xxx")

response = client.messages.create(
    model="claude-opus-4-6",
    max_tokens=1000,
    messages=[
        {"role": "user", "content": "你好"}
    ]
)
```

### 与 OpenAI 的关键差异

| 维度 | OpenAI | Anthropic |
|------|--------|-----------|
| 请求端点 | `chat/completions` | `messages` |
| 角色体系 | system/user/assistant/tool | user/assistant（无独立 system 角色） |
| 消息结构 | `{"role": ..., "content": ...}` | 同上，但 content 可包含多个 block |
| 工具调用 | `tools` 参数 | `tools` 参数格式不同 |
| 流式 | `stream=True` | `stream=True` |

### Anthropic 的独特优势

| 特性 | 说明 |
|------|------|
| **Extended Thinking** | 暴露模型推理过程，用于调试和审计 |
| **Memory Tool（beta）** | 跨会话持久状态 |
| **Effort 参数** | 单参数控制计算投入（低/中/高） |
| **Constitutional AI** | 监管行业（医疗/金融）最看重 |
| **零数据保留** | API 调用默认不用于训练 |

---

## Gemini 兼容格式

### 原生格式特点

Google Gemini 的原生格式与 OpenAI 和 Anthropic 差异较大，需要传 `contents` 而非 `messages`。

### OpenAI 兼容模式

2025 年 4 月，Google 在 Vertex AI 上推出了 OpenAI 兼容端点：

```python
from openai import OpenAI

# 只需改 base_url 和 api_key
client = OpenAI(
    base_url="https://.../openai/chat/completions",
    api_key="gemini-api-key"
)

# 后续代码完全复用
response = client.chat.completions.create(
    model="gemini-2.0-flash",
    messages=[{"role": "user", "content": "你好"}]
)
```

### 重要：兼容模式的限制

> **以下内容非常重要**——兼容模式不是完全兼容。

| 限制 | 说明 |
|------|------|
| **安全设置不可用** | `safety_settings` 参数在兼容模式下不工作 |
| **工具调用不稳定** | Gemini 1.5 Pro 完全拒绝工具调用；2.0 Flash 可能返回 400 |
| **知识库 RAG 失败** | 在 Dify 等平台中兼容模式下知识库检索失败 |
| **可靠性** | ai.google.dev API 在高峰期可能不稳定 |
| **Google 推荐** | 生产环境使用**原生 SDK** |

---

## 三种格式核心差异对比

| 维度 | OpenAI | Anthropic | Gemini |
|------|--------|-----------|--------|
| **端点** | `/chat/completions` | `/messages` | `/models/{model}:generateContent` |
| **request 消息字段** | `messages` | `messages` | `contents` |
| **system prompt** | role="system" | 外部配置参数 | `system_instruction` |
| **多轮对话格式** | 消息列表 | 消息列表 | contents 列表 |
| **工具调用** | `tools` | `tools` | `tools` / `function_declarations` |
| **兼容模式** | 原生 | `/v1/messages`（2025.5） | Vertex AI（2025.4） |
| **兼容模式成熟度** | -- | 较成熟 | Beta，部分功能缺失 |

---

## 兼容格式实操

### 用 OpenAI SDK 调用 Gemini

```python
from openai import OpenAI

# Gemini 的 OpenAI 兼容模式
client = OpenAI(
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/",
    api_key="GEMINI_API_KEY"
)

response = client.chat.completions.create(
    model="gemini-2.0-flash",
    messages=[{"role": "user", "content": "用 Python 写一个快速排序"}]
)

# 业务代码完全复用
print(response.choices[0].message.content)
```

**业务代码完全复用**——只要不动用特殊功能（工具调用、安全设置），基础的问答流程可以无缝切换。

### 通用抽象层：一次编写，处处运行

如果你需要管理多个模型、多个厂商，可以考虑现成的统一抽象层：

| 工具 | 类型 | 特点 |
|------|------|------|
| **LiteLLM** | 开源 Python SDK / 代理 | 100+ 模型，自托管，负载均衡/重试/fallback/成本追踪 |
| **OpenRouter** | 托管 SaaS 网关 | 500+ 模型，零基础设施，统一计费（约 5% 手续费） |
| **Bifrost (Maxim AI)** | 托管网关 | 11us 超低延迟，Rust 后端 |

```python
# LiteLLM 示例
from litellm import completion

# 同一函数，改 model 名即可切换
response = completion(
    model="openai/gpt-5",
    messages=[{"role": "user", "content": "你好"}]
)

response = completion(
    model="anthropic/claude-opus-4-6",
    messages=[{"role": "user", "content": "你好"}]
)
```

---

## 第三章节总结与互动

| 厂商 | 定位 | 特点 |
|------|------|------|
| **OpenAI** | 事实标准 | 生态最丰富，其他厂商在兼容 |
| **Anthropic** | 独立体系 + 后发兼容 | 有独特优势（Thinking、Memory、Constitutional AI） |
| **Gemini** | 灵活兼容 | 兼容模式 Beta，有功能限制 |

### 核心理念

- **一次编写，处处运行**——通过兼容格式或抽象层降低切换成本
- **避免锁定**——但同时对兼容模式的限制保持清醒
- **生产环境优先用原生 SDK**——兼容模式适合评估和迁移测试

---

# 课程复盘与课后检验

## 核心知识点回顾

1. **RESTful API**：资源驱动、HTTP 方法语义、无状态、统一接口
2. **JSON 数据格式**：轻量、跨语言、严格语法（无注释、无双引号、无尾随逗号）
3. **大模型兼容格式**：OpenAI 主导、Anthropic 独立、Gemini 追赶

## 课后实战练习

### Task 1：API 设计

从零设计一套“图书管理系统”RESTful API，至少覆盖图书资源的增删改查（CRUD），需说明设计思路。

**提示**：
- 用名词复数命名资源
- 正确使用 GET/POST/PUT/PATCH/DELETE
- 使用标准 HTTP 状态码
- 考虑分页和过滤

### Task 2：代码实现

编写 Python 代码，调用任一主流大模型 API，将请求参数序列化为 JSON，并正确解析返回的 JSON 响应数据。

**提示**：
- 使用 `json.dumps()` 和 `json.loads()`
- 注意 `ensure_ascii=False` 处理中文
- 正确处理响应中的 `choices[0].message.content`
