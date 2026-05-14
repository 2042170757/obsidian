---
title: HTTP打怪升级手册
date: 2026-05-05
tags: [HTTP, 网络协议, Web开发, 学习笔记]
aliases: [HTTP手册, HTTP指南]
---

# HTTP 网络技术完整指南

## 概述
本文是 HTTP 网络技术的综合指南，涵盖从协议基础到实时通信、跨域安全的完整知识体系。基于 RFC 标准、MDN Web Docs、WHATWG 规范等权威来源编写，每个难懂的概念都配有生动的比喻帮助理解。

## 相关概念
- [[wiki/Network/HTTP基础]] - HTTP 协议基础概览
- [[wiki/Network/HTTP方法]] - HTTP 请求方法详解
- [[wiki/Network/HTTP状态码]] - HTTP 状态码分类与排查
- [[wiki/Network/实时通信技术]] - SSE、WebSocket 等实时技术
- [[wiki/Network/CORS跨域]] - 跨域资源共享机制

---

# 第一部分：HTTP 基础

## 1. 什么是 HTTP

### 定义

HTTP（HyperText Transfer Protocol，超文本传输协议）是一种用于分布式、协作式、超媒体信息系统的应用层协议。它是 Web 上数据通信的基础，自 1990 年引入以来一直是万维网的主要信息传输协议。

> **比喻：餐厅点餐**
> 想象你走进一家餐厅。你是**客户端**（顾客），餐厅是**服务器**（厨房）。你拿起菜单点餐（发送请求），厨房收到后做菜并端给你（返回响应）。每次你想加菜，都得重新跟服务员说（新请求）——厨房不会记住你之前点了什么。这就是 HTTP 的请求-响应模型。

### 客户端-服务器通信模型

HTTP 采用请求-响应（Request-Response）模型：
- **客户端**（通常是浏览器）发送请求
- **服务器**接收请求并返回响应
- 通信由客户端主动发起，服务器被动响应

### 无状态核心特性

HTTP 是**无状态协议**（Stateless Protocol）：每个请求都是独立的，服务器不会保留之前请求的任何信息。

> **比喻：失忆服务员**
> 想象一个服务员有严重的失忆症。你第一次跟他说"我要一杯咖啡"，他记下来送到厨房，做好端给你。然后你再说"加一份三明治"，他已经不记得你点了咖啡——你必须重新说"我要一杯咖啡和一份三明治"。每次对话都是全新的，这就是 HTTP 的无状态特性。

RFC 9110 Section 9.2.1 明确指出：

> "All REST interactions are stateless. That is, each request contains all of the information necessary for a connector to understand the request, independent of any requests that may have preceded it."
>
> **译文**：所有的 REST 交互都是无状态的。也就是说，每一个请求都包含了连接器理解该请求所需的全部信息，而不依赖于此前可能已经发生的任何请求。

这意味着：
- 服务器不记住客户端的之前请求
- 如果需要状态管理，需借助 Cookie、Session、Token 等机制
- 无状态设计使得服务器可以并行处理请求，提高可扩展性

### 端口

| 协议 | 默认端口 | 说明 |
|------|---------|------|
| HTTP | 80 | 明文传输 |
| HTTPS | 443 | TLS/SSL 加密传输 |

> **比喻：HTTPS（电话亭加密）**
> 普通 HTTP 就像在公共场所大声说话，周围人都能听到。HTTPS 则像你走进一个隔音的电话亭——通话内容被加密保护，外面的人听不到。TLS/SSL 就是那个电话亭的隔音墙。

### 常见误区

- **误区**：HTTP 是一个"连接"协议。**正解**：HTTP 是应用层协议，建立在 TCP 之上（HTTP/3 则基于 QUIC/UDP）
- **误区**：无状态意味着不能做登录功能。**正解**：无状态是协议层特性，应用层可通过 Cookie/Session 模拟状态

### 来源
- RFC 9110: HTTP Semantics — https://rfc-editor.org/rfc/rfc9110
- MDN: HTTP overview — https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview

---

## 2. HTTP 的工作过程

### 完整流程

一次完整的 HTTP 请求-响应过程：

```
1. DNS 域名解析
   浏览器缓存 → 系统缓存 → 路由器缓存 → DNS 服务器

2. TCP 连接建立（三次握手）
   客户端 → SYN → 服务器
   服务器 → SYN+ACK → 客户端
   客户端 → ACK → 服务器

3. 请求发送
   客户端发送 HTTP 请求报文（请求行 + 请求头 + 请求体）

4. 响应接收
   服务器处理请求并返回 HTTP 响应报文（状态行 + 响应头 + 响应体）

5. 连接关闭或复用
   - HTTP/1.0: 默认短连接，每次请求后关闭
   - HTTP/1.1: 默认长连接（Connection: keep-alive），可复用
   - HTTP/2: 多路复用，在单个 TCP 连接上并行传输多个请求
   - HTTP/3: 基于 QUIC（UDP），彻底解决队头阻塞问题
```

### DNS 域名解析过程

> **比喻：查快递地址**
> DNS 解析就像你不知道朋友家的具体地址，需要一步步查：
> 1. 先翻**自己手机里的通讯录**（浏览器/系统缓存）——找到了就直接用
> 2. 没找到，问**小区物业**（本地 DNS 服务器）——物业帮你查
> 3. 物业也不知道，就问**街道办事处**（根 DNS）——"这个人在哪个区？"
> 4. 街道说"在海淀区"，物业再去问**海淀区公安局**（顶级域 DNS）——"这个人在哪个小区？"
> 5. 公安局说"在某某小区"，物业最后问**小区物业**（权威 DNS）——"具体几栋几号？"
> 6. 拿到完整地址，返回给你

完整的 DNS 递归解析流程：

```
客户端（浏览器）
  │
  ├── 1. 浏览器缓存 → 有记录？→ 直接返回 IP
  │
  ├── 2. 操作系统缓存（hosts 文件 + DNS 缓存）→ 有记录？→ 直接返回 IP
  │
  ├── 3. 路由器缓存 → 有记录？→ 直接返回 IP
  │
  └── 4. 本地 DNS 服务器（ISP 提供，如 8.8.8.8 / 114.114.114.114）
           │
           │  本地 DNS 没有缓存，开始递归查询：
           │
           ├──→ 根 DNS 服务器（全球 13 组，如 a.root-servers.net）
           │      回答：".com 顶级域的地址是 x.x.x.x"
           │
           ├──→ 顶级域 DNS 服务器（.com / .cn / .org）
           │      回答："example.com 的权威 DNS 是 y.y.y.y"
           │
           └──→ 权威 DNS 服务器（域名注册时配置的）
                  回答："www.example.com 的 IP 是 93.184.216.34"
           │
           本地 DNS 将最终结果缓存并返回给客户端
```

**逐级查询示例**（以 `www.example.com` 为例）：

| 步骤 | 查询对象 | 问题 | 回答 |
|------|---------|------|------|
| 1 | 根 DNS | "www.example.com 的 IP 是什么？" | "我不知道，但 .com 归这个顶级域服务器管 → 198.41.0.4" |
| 2 | .com 顶级域 DNS | "www.example.com 的 IP 是什么？" | "我不知道，但 example.com 归这个权威服务器管 → 205.251.195.131" |
| 3 | example.com 权威 DNS | "www.example.com 的 IP 是什么？" | "IP 是 93.184.216.34" |

**TTL（Time To Live）与缓存**：

- 每条 DNS 记录都有 TTL 值（单位：秒），告诉缓存方"这条记录有效期多久"
- 浏览器、操作系统、路由器、本地 DNS 都会缓存结果，TTL 过期后才重新查询
- 常见 TTL：5 分钟 ~ 24 小时，变更 DNS 记录时需要等 TTL 过期才能全局生效

**查询方式对比**：

| 方式 | 说明 | 类比 |
|------|------|------|
| **递归查询** | 客户端只问一次，DNS 服务器负责一路追查到底 | "你帮我查到底，我等结果" |
| **迭代查询** | 客户端每次得到一个"下一步该问谁"的提示，自己逐级问 | "你告诉我问谁，我自己去问" |

- 客户端 → 本地 DNS：通常是**递归查询**（客户端说"你帮我查"）
- 本地 DNS → 根/顶级域/权威 DNS：通常是**迭代查询**（本地 DNS 自己逐级追查）

### 各协议工作在哪一层

> **比喻：寄快递的分工**
> 把一次 HTTP 请求想象成寄快递：
> - **应用层（HTTP/DNS）**= 你写信的内容和收件地址——你要说什么、寄给谁
> - **传输层（TCP）**= 快递公司的打包和签收确认——确保包裹不丢不坏、按顺序送达
> - **网络层（IP）**= 快递公司的路线规划——从北京到上海走哪条路
> - **数据链路层**= 每一站的短途运输——从快递站到快递站的具体车辆
> - **物理层**= 公路和铁路本身——实际的运输通道

DNS、TCP、HTTP 分别工作在不同层次，各司其职：

| 协议 | 所在层次 | 核心职责 | 一句话概括 |
|------|---------|---------|-----------|
| **DNS** | 应用层 | 域名解析为 IP 地址 | "查电话簿找号码" |
| **HTTP** | 应用层 | 定义数据的语义和格式 | "写信的内容和格式" |
| **TCP** | 传输层 | 可靠的端到端数据传输 | "确保信送到、不丢不乱" |

**层次模型对照**：

```
┌─────────────┬──────────────────────────────────────┐
│   应用层     │  HTTP / HTTPS / DNS / FTP / SMTP     │  ← 你要做什么
├─────────────┼──────────────────────────────────────┤
│   传输层     │  TCP / UDP                           │  ← 怎么可靠送达
├─────────────┼──────────────────────────────────────┤
│   网络层     │  IP (IPv4 / IPv6)                    │  ← 送到哪里
├─────────────┼──────────────────────────────────────┤
│  数据链路层   │  Ethernet / Wi-Fi                    │  ← 局部怎么传
├─────────────┼──────────────────────────────────────┤
│   物理层     │  光纤 / 网线 / 无线电波               │  ← 实际信号
└─────────────┴──────────────────────────────────────┘
```

**为什么 DNS 和 HTTP 都在应用层？**

因为它们都是"应用程序之间约定的通信格式"，而不是"数据怎么传输"的问题：
- **DNS** 解决的是"域名对应哪个 IP"——这是应用层的查询约定
- **HTTP** 解决的是"请求什么资源、用什么格式"——这是应用层的语义约定
- **TCP** 解决的是"数据怎么可靠地从 A 传到 B"——这是传输层的工作

一个 HTTP 请求在发送时，数据会从应用层逐层向下封装：
```
应用层（HTTP）：  GET /index.html HTTP/1.1
    ↓ + TCP 头
传输层（TCP）：   [源端口 | 目标端口 | 序列号 | ...] + HTTP 数据
    ↓ + IP 头
网络层（IP）：    [源 IP | 目标 IP | ...] + TCP 段
    ↓ + 帧头
数据链路层：      [MAC 地址 | ...] + IP 包
    ↓
物理层：          01010110...（电信号/光信号）
```

> **比喻：俄罗斯套娃**
> 数据封装就像俄罗斯套娃：HTTP 数据被包进 TCP 段，TCP 段被包进 IP 包，IP 包被包进链路帧。每一层只关心自己的部分，拆开一层才能看到下一层。

### 三次握手详解

> **比喻：打电话确认**
> 三次握手就像两个人打电话确认对方能听到：
> 1. 你拨电话说"喂，你能听到我说话吗？"（SYN）
> 2. 对方回答"能听到，你能听到我吗？"（SYN+ACK）
> 3. 你说"我也能听到你，我们开始聊吧"（ACK）
>
> 双方都确认了对方能听到，通话才正式开始。

| 步骤  | 方向      | 内容                 | 状态变化             |
| --- | ------- | ------------------ | ---------------- |
| 1   | 客户端→服务器 | SYN seq=x          | 客户端进入 SYN_SENT   |
| 2   | 服务器→客户端 | SYN seq=y, ACK=x+1 | 服务器进入 SYN_RCVD   |
| 3   | 客户端→服务器 | ACK=y+1            | 双方进入 ESTABLISHED |

### 常见误区

- **误区**：HTTP 协议本身负责建立 TCP 连接。**正解**：TCP 连接由操作系统网络栈完成，HTTP 只是在已建立的 TCP 连接上发送应用数据
- **误区**：每次 HTTP 请求都会经历完整的 DNS + TCP 握手过程。**正解**：DNS 和 TCP 连接都有缓存和复用机制

### 来源
- RFC 9110 — https://rfc-editor.org/rfc/rfc9110
- RFC 9293: TCP — https://rfc-editor.org/rfc/rfc9293

---

## 3. URL 完整结构

### 语法定义（RFC 3986）

> **比喻：快递地址**
> URL 就像一个完整的快递地址：
> - **协议**（https）= 快递公司（顺丰/圆通）
> - **域名**（www.example.com）= 收件人所在小区
> - **端口**（:443）= 具体楼栋号
> - **路径**（/api/v1/users）= 单元楼 + 楼层 + 房间号
> - **查询参数**（?page=1）= 附加说明"请放门口"
> - **锚点**（#profile）= 你自己在包裹上做的标记，快递员看不到

```
scheme://[userinfo@]host[:port]/path[?query][#fragment]
```

### 各组成部分详解

| 组成部分 | 说明 | 示例 |
|---------|------|------|
| **协议（Scheme）** | 传输协议 | `https`、`http`、`ftp` |
| **域名/端口** | 服务器地址 | `www.example.com:443` |
| **路径（Path）** | 资源在服务器上的位置 | `/api/v1/users` |
| **查询参数（Query）** | 以 `?` 开头，`&` 分隔的键值对 | `?page=1&size=10` |
| **锚点（Fragment）** | 以 `#` 开头，指定页面内锚点 | `#section2` |

### 完整示例

```
https://www.example.com:443/api/v1/users?page=1&size=10#profile
  ^       ^                ^   ^            ^              ^
  |       |                |   |            |              |
协议    域名             端口 路径       查询参数        锚点
```

### 各部分关键细节

- **协议**：`https` 通过 TLS 加密，`http` 明文传输。现代浏览器对 `http` 站点标记为"不安全"
- **域名**：不区分大小写。端口可省略（HTTP 默认 80，HTTPS 默认 443）
- **路径**：区分大小写。`/API` 和 `/api` 可能指向不同资源
- **查询参数**：键值对之间用 `&` 连接。值中的特殊字符需 URL 编码（如空格编码为 `%20` 或 `+`）
- **锚点**：仅在客户端（浏览器）使用，**不会发送到服务器**

### 常见误区

- **误区**：URL 和 URI 是同一个东西。**正解**：URL 是 URI 的子集。URI 可以是 URL（定位资源）或 URN（命名资源）
- **误区**：锚点会随请求发送到服务器。**正解**：Fragment 部分由浏览器处理，不会出现在 HTTP 请求中

### 来源
- RFC 3986: URI Generic Syntax — https://rfc-editor.org/rfc/rfc3986
- MDN: URL — https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Identifying_resources_on_the_Web

---

## 4. HTTP 请求核心组成

HTTP 请求由四部分组成：

### Header（请求头）

传递请求的元数据，格式为 `Key: Value`，每行一个头字段。核心请求头包括：

| 头字段 | 作用 | 示例 |
|--------|------|------|
| Host | 指定目标主机名（HTTP/1.1 必需） | `Host: www.example.com` |
| User-Agent | 标识客户端软件信息 | `User-Agent: Mozilla/5.0 ...` |
| Accept | 告知服务器客户端可接受的响应格式 | `Accept: application/json` |
| Content-Type | 指定请求体的数据格式 | `Content-Type: application/json` |
| Authorization | 传递认证凭据 | `Authorization: Bearer <token>` |
| Cookie | 传递存储的 Cookie | `Cookie: session_id=abc123` |

**常见误区**：
- 自定义头不再需要 `X-` 前缀（RFC 6648 已弃用）
- HTTP/2 中头字段名必须小写传输（HTTP/1.x 中大小写不敏感）

### Body（请求体）

携带需要发送给服务器的数据，主要用于 POST、PUT、PATCH 请求。常见格式：

| Content-Type | 格式 | 适用场景 |
|-------------|------|---------|
| `application/json` | JSON 字符串 | API 调用（最常用） |
| `application/x-www-form-urlencoded` | key=value&key2=value2 | HTML 表单提交 |
| `multipart/form-data` | 分段二进制数据 | 文件上传 |
| `text/plain` | 纯文本 | 简单文本提交 |

### 常见误区

- **误区**：GET 请求不能有 Body。**正解**：RFC 9110 并未禁止 GET 请求携带 Body，但语义上不推荐，且大多数服务器实现会忽略它
- **误区**：Content-Type 只用于请求体。**正解**：Content-Type 既可用于请求头（描述请求体格式），也可用于响应头（描述响应体格式）

### 来源
- RFC 9110, Section 6.3 (Header Fields) — https://rfc-editor.org/rfc/rfc9110#section-6.3
- MDN: HTTP headers — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers

---

# 第二部分：请求方法

## 5. 请求方法 Method

### 方法定义

HTTP 方法定义了请求的目的和语义。RFC 9110 Section 9 定义了标准方法。

> **比喻：图书馆操作**
> 把 HTTP 方法想象成你在图书馆可以做的操作：
> - **GET**（查阅）：你去书架上看一本书的内容，看完放回去，书没变。你可以反复看，结果一样。
> - **POST**（提交申请）：你去柜台提交一份新书采购申请。每次提交都是一个新的申请，重复提交会产生重复的申请单。
> - **PUT**（替换整本书）：你把书架上整本书的内容全部替换成新版本。不管你替换几次，最终都是那个新版本。
> - **DELETE**（下架）：你要求把某本书从书架上下架。第一次下架成功，第二次书已经不在了（返回 404），但书架状态没变。
> - **PATCH**（修改部分章节）：你只修改书中的某个章节，而不是整本书。通常不是幂等的——如果你说"给第三章加一句话"，执行两次就加了两句。

### 两个核心属性

- **Safe（安全）**：方法不会修改服务器上的资源。GET、HEAD、OPTIONS、TRACE 是安全的
- **Idempotent（幂等）**：多次相同请求的效果与一次请求相同。GET、PUT、DELETE、HEAD、OPTIONS、TRACE 是幂等的

### 标准方法一览

| 方法      | 语义                | Safe | Idempotent | 有 Body   |
| ------- | ----------------- | ---- | ---------- | -------- |
| GET     | 获取资源的表示           | Yes  | Yes        | 不推荐      |
| HEAD    | 获取资源的元数据（无 Body）  | Yes  | Yes        | No       |
| POST    | 提交数据，创建资源或触发处理    | No   | No         | Yes      |
| PUT     | 替换完整资源            | No   | Yes        | Yes      |
| DELETE  | 删除资源              | No   | Yes        | Optional |
| PATCH   | 部分更新资源            | No   | No*        | Yes      |
| OPTIONS | 获取资源支持的通信选项       | Yes  | Yes        | Optional |
| TRACE   | 沿到目标资源的路径执行消息环回测试 | Yes  | Yes        | No       |
| CONNECT | 建立到目标服务器的隧道       | No   | No         | No       |

> *PATCH 的幂等性见下方详解

### 常见误区

- **误区**：RESTful 规范要求严格用 PUT 做更新、POST 只能做创建。**正解**：Roy Fielding 本人明确表示 REST 论文中没有提到 CRUD，POST 的定义是"这个操作不值得标准化"
- **误区**：DELETE 之后资源就一定被删除了。**正解**：DELETE 语义是"请求删除"，服务器可以选择软删除或拒绝删除

### 来源
- RFC 9110, Section 9: Methods — https://rfc-editor.org/rfc/rfc9110#section-9
- IANA HTTP Method Registry — https://www.iana.org/assignments/http-methods/
- MDN: HTTP request methods — https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods

---

## 6. GET 方法

### 语义定义

RFC 9110 Section 9.3.1：

> "The GET method requests transfer of a current selected representation for the target resource."

### 三个核心属性

| 属性 | GET | 含义 |
|------|-----|------|
| Safe | Yes | 不应修改服务器资源 |
| Idempotent | Yes | 多次请求结果相同，可安全重试 |
| Cacheable | Yes | 响应可被缓存 |

### URL 参数传递

GET 请求的参数通过 URL 查询字符串传递：

```javascript
// 浏览器中发起 GET 请求
const response = await fetch('/api/users?page=1&size=10&sort=name');
const users = await response.json();
```

- 参数出现在 `?` 之后，键值对用 `&` 连接
- 参数值需进行 URL 编码（如 `name=John Doe` → `name=John%20Doe`）
- URL 长度有限制：浏览器通常限制 2048-8192 字符（因浏览器而异）

### 常见误区

- **误区**：GET 请求绝对不能有副作用。**正解**：Safe 是语义约定，不是强制约束。服务器实现可以有副作用（如记录日志），但不应有用户可见的状态变化
- **误区**：GET 比 POST 更快。**正解**：在同等条件下性能差异微乎其微，主要区别在语义和缓存行为上

### 来源
- RFC 9110, Section 9.3.1: GET — https://rfc-editor.org/rfc/rfc9110#section-9.3.1

---

## 7. POST 方法

### 语义定义

RFC 9110 Section 9.3.3：

> "The POST method requests that the target resource process the representation enclosed in the request according to the resource's own specific semantics."

### 非安全非幂等特性

| 属性 | POST | 含义 |
|------|------|------|
| Safe | **No** | 可能修改服务器资源 |
| Idempotent | **No** | 多次提交可能产生不同结果 |
| Cacheable | Conditional | 仅当响应包含明确的新鲜度信息时可缓存 |

### 代码示例

```javascript
// Node.js 中发起 POST 请求
const response = await fetch('/api/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer <token>'
  },
  body: JSON.stringify({
    name: '张三',
    email: 'zhangsan@example.com'
  })
});

const newUser = await response.json();
console.log(response.status); // 201 Created
```

**与 GET 的关键区别**：POST 参数在 Body 中，没有大小限制（仅受服务器配置限制）。

### 典型使用场景

1. **创建资源**：`POST /api/users` 创建新用户
2. **表单提交**：HTML form 的默认提交方法
3. **文件上传**：`multipart/form-data` 上传文件
4. **触发处理**：如发送邮件、触发计算任务
5. **不适合其他方法的操作**：Roy Fielding 说 POST 的通用用途是"this action isn't worth standardizing"

### 常见误区

- **误区**：POST 不能用于更新操作。**正解**：POST 的语义是"让服务器处理"，完全可以用于更新
- **误区**：重复 POST 一定产生重复数据。**正解**：幂等性取决于服务器实现。可以设计为幂等（如用唯一 ID 去重）

### 来源
- RFC 9110, Section 9.3.3: POST — https://rfc-editor.org/rfc/rfc9110#section-9.3.3
- Roy Fielding 博客 — https://roy.gbiv.com/untangled/2009/03

---

## 8. PUT 与 DELETE

### PUT — 更新完整资源

**语义定义**（RFC 9110 Section 9.3.4）：

> "The PUT method requests that the state of the target resource be created or replaced with the state defined by the representation enclosed in the request message content."

| 属性 | PUT |
|------|-----|
| Safe | No |
| Idempotent | **Yes** |
| Cacheable | No |

```javascript
// PUT 请求：替换整个用户资源
const response = await fetch('/api/users/123', {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    name: '张三',
    email: 'new@example.com',
    age: 30
    // 必须提供完整资源，缺失字段视为未设置
  })
});
```

### DELETE — 删除资源

**语义定义**（RFC 9110 Section 9.3.5）：

> "The DELETE method requests that the origin server remove the association between the target resource and its current functionality."

| 属性 | DELETE |
|------|--------|
| Safe | No |
| Idempotent | **Yes** |
| Cacheable | No |

```javascript
// DELETE 请求
const response = await fetch('/api/users/123', {
  method: 'DELETE'
});
console.log(response.status); // 204 No Content
```

### PUT vs PATCH 对比

| 维度 | PUT | PATCH |
|------|-----|-------|
| 更新方式 | 完整替换 | 部分修改 |
| 幂等性 | 是 | 否（规范定义）* |
| Body 内容 | 完整资源表示 | 描述变更的指令 |
| 缺失字段 | 视为未设置/删除 | 不影响 |

### 常见误区

- **误区**：PUT 的幂等性意味着不会产生任何副作用。**正解**：幂等性只保证多次请求的最终状态相同，不阻止服务器记录日志等副作用
- **误区**：DELETE 幂等意味着返回码一定相同。**正解**：第一次 DELETE 可能返回 200/204，第二次可能返回 404，但资源状态不变，所以语义上仍然幂等

### 来源
- RFC 9110, Section 9.3.4: PUT — https://rfc-editor.org/rfc/rfc9110#section-9.3.4
- RFC 9110, Section 9.3.5: DELETE — https://rfc-editor.org/rfc/rfc9110#section-9.3.5

---

## 9. PATCH 的幂等性详解

### RFC 5789 的双重定义

RFC 5789 对 PATCH 的幂等性有一个重要的双重定义：

**原文定义（非幂等）**：

> "PATCH is neither safe nor idempotent as defined by [RFC2616], Section 9.1."

**但同时指出（可设计为幂等）**：

> "A PATCH request can be issued in such a way as to be idempotent, which also helps prevent bad outcomes from collisions between two PATCH requests on the same resource in a similar time frame."

### 关键理解

- RFC 将 PATCH 定义为**默认非幂等**——这是它的规范属性
- 但 RFC 明确允许特定实现使 PATCH 成为幂等的
- IANA 注册表中 PATCH 标记为 `Idempotent: no`

### 幂等 vs 非幂等的 PATCH 示例

```javascript
// 非幂等的 PATCH（增量操作）—— 执行两次结果不同
await fetch('/api/cart/123', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ quantity: '+1' })  // 第一次: 1→2, 第二次: 2→3
});

// 幂等的 PATCH（设置操作）—— 执行多次结果相同
await fetch('/api/users/123', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: '张三' })  // 无论执行几次，name 都是"张三"
});
```

### 实践建议

- 不要依赖 PATCH 的默认幂等性
- 如果需要幂等更新，使用 PUT 或添加 `If-Match` 条件头
- 在 API 文档中明确说明每个 PATCH 端点是否幂等
- 许多实际 API 的 PATCH 操作（如设置字段值）在实践中是幂等的，但应视为实现细节而非规范保证

### 来源
- RFC 5789: PATCH Method for HTTP — https://rfc-editor.org/rfc/rfc5789
- IANA HTTP Method Registry — https://www.iana.org/assignments/http-methods/

---

## 10. RESTful 方法最佳实践

### 核心思想

REST（Representational State Transfer）由 Roy Fielding 在其 2000 年的博士论文中提出。它是一种**架构风格**，而非具体的 API 设计规范。

### HTTP 方法与 CRUD 操作映射（业界惯例）

| CRUD 操作 | HTTP 方法 | URL 示例 | 典型状态码 |
|-----------|----------|---------|-----------|
| Create（创建） | POST | `POST /api/users` | 201 Created |
| Read（读取） | GET | `GET /api/users/123` | 200 OK |
| Update（更新） | PUT / PATCH | `PUT /api/users/123` | 200 OK / 204 No Content |
| Delete（删除） | DELETE | `DELETE /api/users/123` | 204 No Content |

**重要澄清**：Roy Fielding 本人明确表示 REST 架构风格本身不规定 HTTP 方法与 CRUD 的映射关系。以上映射是业界的最佳实践惯例，而非 REST 的硬性要求。

> "Search my dissertation and you won't find any mention of CRUD or POST." — Roy Fielding, 2009

### 常见误区

- **误区**：RESTful API 必须严格一对一映射 HTTP 方法和 CRUD。**正解**：REST 是架构风格，方法使用应遵循 HTTP 语义定义即可
- **误区**：使用 POST 做更新就是"不 RESTful"。**正解**：关键在于不违反方法的语义定义

### 来源
- Roy Fielding 博士论文 — https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm
- Roy Fielding 博客（2009） — https://roy.gbiv.com/untangled/2009/03

---

# 第三部分：状态码

## 11. HTTP 状态码分类

### 五大类（RFC 9110 Section 15）

> **比喻：餐厅回应**
> 状态码就像餐厅对你点餐请求的回应方式：
> - **1xx**（"稍等一下"）：服务员说"我听到了，正在处理"
> - **2xx**（"好的，这是你要的"）：菜做好了端上来了
> - **3xx**（"你要找的不在这里"）：服务员说"这道菜在隔壁分店有"
> - **4xx**（"你的问题"）：你点了一道菜单上没有的菜（404），或者你没带会员卡（401）
> - **5xx**（"厨房的问题"）：厨房着火了（500），或者厨师请假了（503）

| 类别 | 范围 | 含义 | 常见数量 |
|------|------|------|---------|
| **1xx** | 100-199 | Informational（信息性） | 4个 |
| **2xx** | 200-299 | Success（成功） | 10个 |
| **3xx** | 300-399 | Redirection（重定向） | 9个 |
| **4xx** | 400-499 | Client Error（客户端错误） | 29个 |
| **5xx** | 500-599 | Server Error（服务器错误） | 12个 |

### 常用状态码详解

#### 1xx 信息性

> **比喻：餐厅的"稍等"**
> 1xx 就像你点完餐后服务员的"稍等"——菜还没做好，但服务员告诉你"我听到了，正在处理"。你不需要做任何动作，继续等就行。浏览器通常会自动处理这些状态码，开发者很少直接接触。

| 状态码     | 名称                  | 说明                    | 典型使用                                 |
| ------- | ------------------- | --------------------- | ------------------------------------ |
| 100     | Continue            | 服务器已收到请求头，请继续发送请求体    | 大文件上传前的"试探"：客户端先发头，确认服务器愿意接收后再发 Body |
| **101** | Switching Protocols | 服务器同意切换协议             | WebSocket 握手（HTTP → WebSocket）       |
| 102     | Processing          | 服务器已收到请求，正在处理（WebDAV） | WebDAV 长时间操作，防止客户端超时                 |
| 103     | Early Hints         | 服务器在最终响应前预先返回资源提示     | 提前推送 CSS/JS 等资源链接，加速页面加载             |



#### 2xx 成功

| 状态码 | 名称 | 说明 | 典型使用 |
|--------|------|------|---------|
| 200 | OK | 请求成功 | GET 成功 |
| 201 | Created | 已创建新资源 | POST 创建成功 |
| 202 | Accepted | 已接受处理，但未完成 | 异步任务提交 |
| 204 | No Content | 成功但无返回内容 | DELETE/PUT 成功 |

#### 3xx 重定向

| 状态码 | 名称 | 说明 | 方法变化 |
|--------|------|------|---------|
| 301 | Moved Permanently | 永久重定向 | 允许变 GET |
| 302 | Found | 临时重定向 | 允许变 GET（历史原因） |
| 303 | See Other | 用 GET 请求另一个 URL | 强制变 GET |
| 304 | Not Modified | 资源未修改，使用缓存 | — |
| 307 | Temporary Redirect | 临时重定向 | **保持原方法** |
| 308 | Permanent Redirect | 永久重定向 | **保持原方法** |

#### 4xx 客户端错误

| 状态码     | 名称                    | 说明         |
| ------- | --------------------- | ---------- |
| **400** | Bad Request           | 请求语法错误     |
| **401** | Unauthorized          | 未认证        |
| **403** | Forbidden             | 无权限        |
| **404** | Not Found             | 资源不存在      |
| **405** | Method Not Allowed    | 方法不被允许     |
| 409     | Conflict              | 资源冲突       |
| 422     | Unprocessable Content | 语义错误（语法正确） |
| **429** | Too Many Requests     | 请求过于频繁     |

#### 5xx 服务器错误

| 状态码     | 名称                    | 说明      |
| ------- | --------------------- | ------- |
| **500** | Internal Server Error | 服务器内部错误 |
| **501** | Not Implemented       | 方法未实现   |
| **502** | Bad Gateway           | 网关错误    |
| **503** | Service Unavailable   | 服务暂不可用  |
| 504     | Gateway Timeout       | 网关超时    |

### 来源
- RFC 9110, Section 15: Status Codes — https://rfc-editor.org/rfc/rfc9110#section-15
- IANA HTTP Status Code Registry — https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml
- MDN: HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

---

## 12. 特殊状态码辨析

### 401 vs 403

| 维度 | 401 Unauthorized | 403 Forbidden |
|------|-----------------|---------------|
| **语义** | 未认证（Unauthenticated） | 无权限（Unauthorized） |
| **客户端身份** | 服务器不知道客户端是谁 | 服务器知道客户端是谁 |
| **是否可重试** | 是，提供认证凭据后重试 | 否，即使认证也无法访问 |
| **响应头** | 必须包含 `WWW-Authenticate` | 不需要 |
| **类比** | "请出示证件" | "你不能进这个地方" |

**RFC 9110 原文定义**：
- 401："The server cannot associate the client with a specific identity"（服务器无法将客户端与特定身份关联）
- 403："the server understood the request but refuses to fulfill it"（服务器理解请求但拒绝执行）

### 405 vs 501

| 维度 | 405 Method Not Allowed | 501 Not Implemented |
|------|----------------------|---------------------|
| **语义** | 服务器**认识**该方法，但目标资源**不支持** | 服务器**不认识**该方法，根本**未实现** |
| **责任方** | 客户端使用了不恰当的方法 | 服务器未实现该方法 |
| **必须响应头** | `Allow`（列出资源支持的方法） | 不要求 |
| **类比** | "我知道 DELETE 是什么，但这个资源不支持删除" | "我不知道 PATCH 是什么方法" |

### 常见误区

- **误区**：收到 401 时应该提示"权限不足"。**正解**：401 表示需要登录，应提示"请先登录"
- **误区**：403 和 404 可以互换使用。**正解**：有些服务器故意对无权限的资源返回 404 以隐藏资源存在，但语义上两者不同
- **误区**：501 表示服务器有 bug。**正解**：501 只表示服务器尚未实现客户端请求的方法

### 来源
- RFC 9110, Section 15.5.2 (401) — https://rfc-editor.org/rfc/rfc9110#section-15.5.2
- RFC 9110, Section 15.5.3 (403) — https://rfc-editor.org/rfc/rfc9110#section-15.5.3
- RFC 9110, Section 15.5.6 (405) — https://rfc-editor.org/rfc/rfc9110#section-15.5.6
- RFC 9110, Section 15.6.2 (501) — https://rfc-editor.org/rfc/rfc9110#section-15.6.2

---

## 13. 状态码问题排查方法

### 系统排查流程

```
Step 1: 确定分类（第一个数字）
├── 1xx → 继续等待或忽略（少见）
├── 2xx → 请求成功，检查业务逻辑
├── 3xx → 检查重定向配置
├── 4xx → 检查客户端请求（最常见）
└── 5xx → 检查服务器端（需查日志）

Step 2: 根据具体状态码排查
4xx:
├── 400 → 检查请求参数格式、JSON 语法
├── 401 → 检查认证 Token / Cookie 是否过期
├── 403 → 检查用户权限、IP 白名单
├── 404 → 检查 URL 路径是否正确
├── 405 → 检查请求方法是否正确
├── 413 → 请求体过大，检查文件大小限制
├── 415 → Content-Type 是否匹配
├── 422 → 请求格式正确但语义有误（如字段校验失败）
└── 429 → 请求频率过高，等待后重试

5xx:
├── 500 → 查看服务器错误日志
├── 502 → 检查上游服务器/网关
├── 503 → 服务器过载或维护中
└── 504 → 上游服务器超时，检查网络或增加超时
```

### 实用排查技巧

1. **使用浏览器 DevTools**：Network 面板查看完整请求/响应头
2. **检查 `Allow` 头**：405 响应会列出支持的方法
3. **检查 `WWW-Authenticate` 头**：401 响应会指定认证方式
4. **使用 `curl -v`**：查看完整的 HTTP 交互
5. **查看 `X-Request-Id`**：很多 API 返回请求 ID，便于追踪

### 常见误区

- **误区**：所有 4xx 都是客户端代码 bug。**正解**：有些 4xx 是正常业务逻辑（如 404 查找不存在的用户、409 资源冲突）
- **误区**：5xx 一定是服务器代码 bug。**正解**：502/504 通常是反向代理或网络问题

### 来源
- MDN: HTTP response status codes — https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status

---

# 第四部分：实时通信技术

## 14. 短轮询（Short Polling）

### 定义

短轮询是客户端以固定间隔反复向服务器发送 HTTP 请求，检查是否有新数据的通信模式。

### 工作原理

```
客户端                    服务器
  |                         |
  |--- GET /api/events --->|  (每 N 秒)
  |<-- {events: []} -------|  (无新数据)
  |                         |
  |--- GET /api/events --->|  (N 秒后)
  |<-- {events: [...]} ----|  (有新数据)
  |                         |
  |--- GET /api/events --->|  (继续轮询)
```

### 代码示例

```javascript
// 短轮询实现
setInterval(async () => {
  const response = await fetch('/api/events');
  const data = await response.json();
  if (data.events.length > 0) {
    handleEvents(data.events);
  }
}, 5000); // 每5秒轮询
```

### 优缺点

| 维度 | 说明 |
|------|------|
| 实现复杂度 | 极低，纯标准 HTTP |
| 网络开销 | **高**：大量无数据的空请求 |
| 实时性 | **低**：延迟 = 0 到轮询间隔 |
| 服务器负载 | **高**：每个客户端每 N 秒一个请求 |
| 适用场景 | 简单应用、实时性要求不高的场景 |

### 常见误区

- **误区**：轮询间隔越短越好。**正解**：过短的间隔会大幅增加服务器负载和带宽消耗，且大部分请求返回空数据
- **误区**：短轮询和长轮询是同一个东西。**正解**：短轮询是请求后立即返回（无论有无数据）；长轮询是服务器保持请求直到有数据才返回

### 来源
- MDN: Server-Sent Events（对比轮询）— https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events

---

## 15. SSE（Server-Sent Events）服务器推送

### 核心定义

> **比喻：收音机广播**
> SSE 就像收音机广播：电台（服务器）持续发送信号，你的收音机（浏览器）调到那个频率就能收到。你不能通过收音机跟电台说话——这是单向的。如果信号断了，收音机会自动重新搜索频率（自动重连），而且它知道上次收到哪个节目（Last-Event-ID），能从断点继续听。

SSE 是一种基于 HTTP 的标准，允许服务器通过一个持久化的 HTTP 连接主动向客户端推送数据。它是**单向数据流**（Server → Client），客户端使用内置的 `EventSource` API 接收数据。

### 关键机制与优势

| 特性 | 说明 |
|------|------|
| 协议 | 基于标准 HTTP（无需升级协议） |
| 方向 | 仅服务器到客户端（单向） |
| 自动重连 | 浏览器内置，断线后自动重连 |
| 事件恢复 | 通过 `Last-Event-ID` 从断点续传 |
| 数据格式 | 仅 UTF-8 文本 |
| 浏览器支持 | 97.23%（2026 年数据） |

### 代码示例

```javascript
// 浏览器端：使用 EventSource 接收 SSE
const eventSource = new EventSource('/api/stream');

eventSource.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('收到消息:', data);
};

eventSource.addEventListener('notification', (event) => {
  const data = JSON.parse(event.data);
  showNotification(data);
});

eventSource.onerror = (error) => {
  console.error('SSE 连接错误:', error);
  // 浏览器会自动重连
};

// 关闭连接
eventSource.close();
```

```javascript
// Node.js 服务端：使用 Express 发送 SSE
app.get('/api/stream', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  let id = 0;
  const interval = setInterval(() => {
    const data = { message: 'Hello', timestamp: Date.now() };
    res.write(`id: ${++id}\n`);
    res.write(`event: notification\n`);
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  }, 1000);

  req.on('close', () => {
    clearInterval(interval);
  });
});
```

### SSE 并发连接数限制（重要）

**HTTP/1.1 环境**：
- 浏览器对同一域名限制**最多 6 个并发连接**（跨所有标签页共享）
- 这个 6 连接限制并非来自 RFC 2616 的直接建议——RFC 2616 实际建议的是 2 个并发连接。6 个是浏览器厂商自行实现的惯例，后来成为事实标准
- Chrome 和 Firefox 都将此问题标记为 "Won't Fix"
- **影响**：如果一个标签页打开了 SSE 连接，会占用 6 个配额中的一个

**HTTP/2 环境**：
- HTTP/2 通过多路复用消除了 6 连接限制
- 默认支持约 **100 个并发流**（可协商调整）
- SSE 仅占用一个复用流，不影响其他请求
- **建议**：生产环境 SSE 应使用 HTTP/2

**规避方案**：
1. 使用 HTTP/2（最佳方案）
2. 使用多个子域名分散连接
3. 使用 SharedWorker 在标签页间共享 SSE 连接
4. 使用 Web Locks API 选举"主标签页"，其他标签页通过 BroadcastChannel 接收

### Wire 格式

```
event: message\n
id: 123\n
data: {"type": "notification", "content": "Hello"}\n
\n
```

### 典型应用场景

- **AI 流式响应**：ChatGPT、Claude 等大模型的流式输出
- **实时通知**：消息推送、订单状态更新
- **实时仪表盘**：股票行情、监控数据
- **活动流**：社交媒体动态

### 常见误区

- **误区**：SSE 和 WebSocket 是同一类技术，可以互换。**正解**：SSE 是单向的（服务端到客户端），WebSocket 是双向的。SSE 基于 HTTP，WebSocket 需要协议升级
- **误区**：SSE 在 HTTP/2 下没有连接限制。**正解**：HTTP/2 消除了 6 连接限制，但服务器端仍需考虑每个客户端持有一个长连接的资源开销
- **误区**：SSE 只能传输简单文本。**正解**：虽然 SSE 只支持 UTF-8 文本，但可以通过 JSON 序列化传输结构化数据

### 来源
- WHATWG HTML Standard: Server-Sent Events — https://html.spec.whatwg.org/server-sent-events.html
- W3C: Server-Sent Events — https://www.w3.org/TR/2021/SPSD-eventsource-20210128/
- MDN: Server-Sent Events — https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events
- Better SSE FAQ — https://matthewwid.github.io/better-sse/reference/faq/

---

## 16. WebSocket 基础

### 定义

> **比喻：对讲机**
> WebSocket 就像对讲机：你按住按钮说话，对方能立刻听到；对方按住按钮说话，你也能立刻听到。双方可以随时互发消息，不需要像打电话那样等对方说完（半双工），也不需要像写信那样来回等（请求-响应）。连接建立后，频道一直开着，直到任何一方挂断。

WebSocket 是一种在单个 TCP 连接上提供**全双工**（Full-Duplex）通信的协议，由 RFC 6455 定义。它允许服务器和客户端在任意时刻主动发送数据，无需等待对方请求。

### 与 HTTP 的关系

| 维度 | HTTP | WebSocket |
|------|------|-----------|
| 通信模式 | 请求-响应（半双工） | 全双工 |
| 连接方式 | 短连接或长连接 | 持久连接 |
| 数据方向 | 客户端发起 | 双向任意时刻 |
| 协议标识 | `http://` / `https://` | `ws://` / `wss://` |
| 建立方式 | 直接连接 | 通过 HTTP Upgrade 握手建立 |
| 头部开销 | 每次请求/响应都有完整头 | 帧头仅 2-14 字节 |

### 代码示例

```javascript
// 浏览器端：WebSocket 客户端
const ws = new WebSocket('wss://example.com/chat');

ws.onopen = () => {
  console.log('连接已建立');
  ws.send(JSON.stringify({ type: 'join', room: 'general' }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('收到消息:', data);
};

ws.onclose = (event) => {
  console.log('连接关闭:', event.code, event.reason);
  // WebSocket 协议不提供自动重连，需要手动实现
  setTimeout(() => reconnect(), 3000);
};

ws.onerror = (error) => {
  console.error('WebSocket 错误:', error);
};

// 发送消息
ws.send(JSON.stringify({ type: 'message', content: 'Hello!' }));
```

```javascript
// Node.js 服务端：使用 ws 库
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  console.log('客户端已连接');

  ws.on('message', (message) => {
    const data = JSON.parse(message);
    // 广播给所有连接的客户端
    wss.clients.forEach((client) => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify(data));
      }
    });
  });

  ws.on('close', () => {
    console.log('客户端断开连接');
  });
});
```

### 常见误区

- **误区**：WebSocket 是 HTTP 的替代品。**正解**：WebSocket 通过 HTTP 握手建立连接，两者互补而非替代
- **误区**：WebSocket 连接可以穿越所有代理和防火墙。**正解**：`ws://` 在企业代理环境中可能失败，`wss://`（基于 TLS）可靠性更高

### 来源
- RFC 6455: The WebSocket Protocol — https://rfc-editor.org/rfc/rfc6455
- MDN: WebSocket API — https://developer.mozilla.org/en-US/docs/Web/API/WebSocket

---

## 17. WebSocket 的握手与传输

### 握手过程（101 协议切换）

WebSocket 连接始于一个 HTTP/1.1 升级请求：

**客户端请求**：
```http
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Origin: http://example.com
```

**服务器响应**：
```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### 握手关键细节

1. **必须是 HTTP/1.1**：HTTP/1.0 不支持连接升级，HTTP/2 使用不同机制（RFC 8441）
2. **必须是 GET 方法**：其他方法无法建立 WebSocket
3. **Sec-WebSocket-Key**：客户端生成的随机 16 字节值（Base64 编码），**不是加密或认证**，仅防止缓存代理误响应
4. **Sec-WebSocket-Accept**：服务器将客户端 Key 与固定 GUID `258EAFA5-E914-47DA-95CA-C5AB0DC85B11` 拼接后取 SHA-1 哈希再 Base64 编码
5. **版本号**：RFC 6455 定义的版本 13 是唯一广泛使用的版本
6. **除了 101 之外的任何状态码**都意味着握手失败。200 OK 意味着服务器把它当作普通 GET 请求处理了

### 全双工帧传输

| 帧类型 | 操作码 | 说明 |
|--------|--------|------|
| Text | 0x1 | UTF-8 文本数据 |
| Binary | 0x2 | 二进制数据 |
| Close | 0x8 | 关闭连接 |
| Ping | 0x9 | 心跳检测 |
| Pong | 0xA | 心跳回复 |

**关键规则**：
- 客户端到服务器的帧**必须**用 4 字节掩码（Mask）加密
- 服务器到客户端的帧**不得**使用掩码
- 掩码不是为了安全，而是防止缓存代理中毒

### 常见误区

- **误区**：Sec-WebSocket-Key 是一种安全机制。**正解**：它仅用于防止缓存代理意外返回缓存的 WebSocket 握手响应
- **误区**：WebSocket 在 wss:// 下也暴露升级过程。**正解**：TLS 加密发生在 WebSocket 握手之前，网络观察者只能看到 TLS 加密流量
- **误区**：WebSocket 连接断开后需要手动重新建立。**正解**：应用层需要实现重连逻辑，WebSocket 协议本身不提供自动重连

### 来源
- RFC 6455, Section 4: Opening Handshake — https://rfc-editor.org/rfc/rfc6455#section-4
- RFC 8441: Bootstrapping WebSockets with HTTP/2 — https://rfc-editor.org/rfc/rfc8441

---

## 17.5 SSE vs WebSocket 选型总结

### 选型速查

**选 SSE 当**：
- 只需要服务器推送数据到客户端
- 想要开箱即用的自动重连和事件续传
- 需要穿越企业代理/防火墙/CDN
- 在 HTTP/2 环境下部署
- 数据量适中，不需要二进制传输

**选 WebSocket 当**：
- 需要客户端频繁向服务器发送数据
- 需要极低延迟（游戏、实时交易）
- 需要传输二进制数据
- 消息频率极高（每秒数百条以上）
- 需要自定义协议（如 GraphQL subscriptions、MQTT over WebSocket）

### 常见误区

- **误区**：WebSocket 一定比 SSE 更好。**正解**：如果只需要服务器推送，SSE 更简单、更可靠（自动重连、标准 HTTP、代理兼容好）
- **误区**：SSE 不能用于生产环境。**正解**：ChatGPT、Claude 等主流产品都使用 SSE 进行 AI 流式输出，生产验证充分
- **误区**：HTTP/2 下 WebSocket 不需要单独连接。**正解**：HTTP/2 支持 WebSocket（RFC 8441），但 WebSocket 仍然需要独立的流，不与其他 HTTP 请求复用

---

# 第五部分：CORS 跨域

## 18. 跨域问题与同源策略

### 同源策略（Same-Origin Policy）

> **比喻：公司门禁**
> 同源策略就像公司的门禁系统。你的工牌只能让你进入自己公司的办公楼（同源）。如果你拿着 A 公司的工牌去 B 公司（跨域），门禁会拒绝你进入。即使 B 公司的大门是开着的（请求到达了服务器），门禁系统（浏览器）也会在你试图刷卡进入时阻止你读取里面的文件（响应数据）。

同源策略是浏览器的核心安全机制。**同源**的定义：协议 + 域名 + 端口三者完全相同。

| URL A | URL B | 是否同源 | 原因 |
|-------|-------|---------|------|
| `http://example.com` | `http://example.com/page` | 同源 | — |
| `http://example.com` | `https://example.com` | **跨域** | 协议不同 |
| `http://example.com` | `http://api.example.com` | **跨域** | 域名不同 |
| `http://example.com:80` | `http://example.com:8080` | **跨域** | 端口不同 |

### 为什么会有跨域拦截

同源策略的目的是**防止恶意网站读取其他网站的敏感数据**。例如：如果没有同源策略，恶意网站 `evil.com` 的 JavaScript 可以向 `bank.com/api/balance` 发送请求（用户已登录的 Cookie 会自动附带），从而窃取银行余额。

### 跨域触发条件

由 JavaScript 发起的以下操作会受同源策略限制：
- `fetch()` 请求
- `XMLHttpRequest` 请求
- Web 字体加载（`@font-face`）
- WebGL 纹理
- 图片/视频的 `drawImage()`

**不受限制的**：
- `<script src="...">`（JSONP 利用的特性）
- `<img src="...">`
- `<link href="...">`
- `<form action="...">` 提交

### 常见误区

- **误区**：CORS 是服务器阻止请求。**正解**：CORS 中请求**已经到达服务器**，服务器**正常处理并返回响应**。是**浏览器**根据 CORS 头决定是否将响应交给 JavaScript
- **误区**：CORS 可以防止 CSRF 攻击。**正解**：CORS 不能阻止请求被发送（简单请求和表单提交不受限），只能阻止 JavaScript 读取响应

### 来源
- Fetch Standard（定义 CORS）— https://fetch.spec.whatwg.org/
- MDN: CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- MDN: Same-origin policy — https://developer.mozilla.org/en-US/docs/Glossary/Same-origin_policy

---

## 19. CORS 的两种请求类型

### 简单请求（Simple Request）

满足以下**所有条件**的请求不会触发预检：

**方法限制**：仅限 GET、HEAD、POST

**头部限制**：仅允许以下安全列表中的头：
- `Accept`
- `Accept-Language`
- `Content-Language`
- `Content-Type`
- `Range`（仅限简单范围头）

**Content-Type 限制**（POST）：仅限以下值：
- `application/x-www-form-urlencoded`
- `multipart/form-data`
- `text/plain`

### 预检请求（Preflight Request）

> **比喻：保安盘问**
> 预检请求就像你去一个安保严格的小区拜访朋友。你到门口时，保安（浏览器）不会直接让你进去，而是先盘问你："你是谁？从哪来？要做什么？带了什么东西？"（发送 OPTIONS 请求）。小区管理处（服务器）审核后告诉保安"这个人可以进，允许他带这些物品"（返回 CORS 头）。确认安全后，保安才放你进去见朋友（发送实际请求）。

当请求不满足简单请求条件时（如使用 PUT/DELETE 方法、自定义头、JSON Content-Type），浏览器会先发送 OPTIONS 预检请求。

### 预检交互三部曲

```
1. 浏览器发送预检请求（OPTIONS）
   OPTIONS /api/users HTTP/1.1
   Host: api.example.com
   Origin: http://frontend.com
   Access-Control-Request-Method: DELETE
   Access-Control-Request-Headers: Authorization, Content-Type

2. 服务器返回预检响应
   HTTP/1.1 204 No Content
   Access-Control-Allow-Origin: http://frontend.com
   Access-Control-Allow-Methods: GET, POST, PUT, DELETE
   Access-Control-Allow-Headers: Authorization, Content-Type
   Access-Control-Max-Age: 86400

3. 浏览器发送实际请求
   DELETE /api/users/123 HTTP/1.1
   Host: api.example.com
   Origin: http://frontend.com
   Authorization: Bearer <token>
```

### CORS Max-Age 的浏览器差异（重要）

`Access-Control-Max-Age` 指定预检结果可被缓存的最大秒数：

| 浏览器 | 最大缓存时间 | 说明 |
|--------|------------|------|
| Firefox | **24 小时**（86400 秒） | — |
| Chromium < v76 | **10 分钟**（600 秒） | 旧版 Chrome、Edge 等 |
| Chromium >= v76 | **2 小时**（7200 秒） | Chrome 76+、新版 Edge |
| Safari/WebKit | **10 分钟**（600 秒） | Safari/WebKit 硬编码上限为 600 秒 |
| **规范默认值** | **5 秒** | 未指定 Max-Age 时的默认行为 |

**修正说明**：Safari/WebKit 的 `Access-Control-Max-Age` 上限为 **600 秒（10 分钟）**，这是 WebKit 源码中的硬编码值。即使服务器返回更大的值（如 86400），Safari 也会将其限制在 600 秒。

**实践建议**：设置 `Access-Control-Max-Age: 86400` 可在所有浏览器上获得最佳缓存效果（Firefox 会使用 86400，Chrome 会自动限制到 7200，Safari 会限制到 600）。

### 常见误区

- **误区**：所有跨域请求都会触发预检。**正解**：简单请求（如 GET 或 `Content-Type: text/plain` 的 POST）不触发预检
- **误区**：预检请求会携带请求体。**正解**：OPTIONS 预检请求没有 Body，只包含 CORS 相关的头信息
- **误区**：预检失败意味着服务器拒绝了请求。**正解**：服务器可能正常处理了 OPTIONS 请求（返回 200），但 CORS 头不匹配，浏览器阻止了实际请求
- **误区**：设置 `Access-Control-Allow-Origin: *` 就能解决所有跨域问题。**正解**：通配符 `*` 不允许携带凭据（Cookie），且不适用于需要认证的场景

### 来源
- Fetch Standard — https://fetch.spec.whatwg.org/
- MDN: CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS
- MDN: Access-Control-Max-Age — https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Max-Age

---

## 20. CORS 的配置与使用

### 服务端响应头配置

#### 核心 CORS 响应头

| 响应头 | 作用 | 示例值 |
|--------|------|--------|
| `Access-Control-Allow-Origin` | 允许的来源 | `https://example.com` 或 `*` |
| `Access-Control-Allow-Methods` | 允许的 HTTP 方法 | `GET, POST, PUT, DELETE` |
| `Access-Control-Allow-Headers` | 允许的请求头 | `Content-Type, Authorization` |
| `Access-Control-Allow-Credentials` | 是否允许携带凭据 | `true` |
| `Access-Control-Max-Age` | 预检缓存时间（秒） | `86400` |
| `Access-Control-Expose-Headers` | 允许 JavaScript 读取的响应头 | `X-Custom-Header` |

#### 常见服务端配置示例

**Nginx**:
```nginx
location /api/ {
    add_header Access-Control-Allow-Origin $http_origin;
    add_header Access-Control-Allow-Methods 'GET, POST, PUT, DELETE, OPTIONS';
    add_header Access-Control-Allow-Headers 'Content-Type, Authorization';
    add_header Access-Control-Allow-Credentials true;
    add_header Access-Control-Max-Age 86400;

    if ($request_method = 'OPTIONS') {
        return 204;
    }
}
```

**Express.js (cors 中间件)**:
```javascript
const cors = require('cors');
app.use(cors({
  origin: 'https://example.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
  maxAge: 86400
}));
```

**手动实现 CORS**:
```javascript
app.use((req, res, next) => {
  const origin = req.headers.origin;
  const allowedOrigins = ['https://example.com', 'https://app.example.com'];

  if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
  }

  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  res.setHeader('Access-Control-Max-Age', '86400');
  res.setHeader('Vary', 'Origin'); // 动态 Origin 时必须设置

  if (req.method === 'OPTIONS') {
    return res.status(204).end();
  }

  next();
});
```

### 常见配置误区

1. **`Access-Control-Allow-Origin: *` 与 `Credentials: true` 冲突**
   - 当 `Allow-Credentials: true` 时，`Allow-Origin` 不能使用通配符 `*`
   - 必须指定具体的源

2. **多域名支持**
   - 不能直接写多个域名：`https://a.com, https://b.com`（不合法）
   - 需要动态检查 `Origin` 请求头，匹配后回显

3. **预检缓存不生效**
   - 确保 `Access-Control-Max-Age` 在预检响应中返回
   - 注意浏览器上限差异（Safari 为 600 秒）

4. **忘记处理 OPTIONS 请求**
   - 某些框架不会自动处理 OPTIONS 预检
   - 需要确保 OPTIONS 请求返回正确的 CORS 头和 204 状态码

5. **Vary: Origin 缺失**
   - 当根据 Origin 动态返回不同 CORS 头时，必须设置 `Vary: Origin`
   - 否则 CDN/缓存可能返回错误的 CORS 头

### 来源
- Fetch Standard — https://fetch.spec.whatwg.org/
- MDN: CORS — https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS

---

# 第六部分：技术选型与课程复盘

## 21. 实时通信技术选型指南

### 技术选型决策树

```
需要实时通信？
├── 只需要服务器向客户端推送？
│   ├── 数据量小、频率低 → SSE (Server-Sent Events)
│   └── 高频推送、需要低延迟 → SSE（HTTP/2）或 WebSocket
│
├── 需要双向通信？
│   ├── 文本消息为主 → WebSocket
│   ├── 二进制数据为主 → WebSocket
│   └── 需要经过严格企业代理 → WebSocket (wss://)
│
└── 实时性要求不高？
    └── 短轮询（简单但效率低）
```

### 技术对比矩阵

| 维度 | 短轮询 | SSE | WebSocket |
|------|-------|-----|-----------|
| **方向** | 客户端→服务器 | 服务器→客户端 | 双向 |
| **协议** | HTTP | HTTP | ws/wss |
| **实时性** | 低（取决于间隔） | 高 | 最高 |
| **连接开销** | 每次新建 | 持久连接 | 持久连接 |
| **消息开销** | 完整 HTTP 头 | ~100-200 字节/消息 | 2-14 字节/帧 |
| **自动重连** | 不需要（每次新建） | 内置 | 需手动实现 |
| **断点续传** | 不支持 | 内置（Last-Event-ID） | 需手动实现 |
| **数据格式** | 任意 | UTF-8 文本 | 文本 + 二进制 |
| **代理兼容** | 完美 | 完美（HTTP） | 可能有问题（ws://） |
| **HTTP/2 支持** | 完美 | 完美（多路复用） | 需 RFC 8441 |
| **浏览器支持** | 所有 | 97.23% | 98.4% |

### 场景选型指南

| 场景 | 推荐技术 | 理由 |
|------|---------|------|
| AI 聊天流式输出 | **SSE** | 单向推送、标准 HTTP、自动重连 |
| 在线客服聊天 | **WebSocket** | 双向通信、低延迟 |
| 实时通知/消息推送 | **SSE** | 简单高效、HTTP 原生支持 |
| 股票实时行情 | **WebSocket** | 高频双向、二进制数据支持 |
| 多人协作编辑 | **WebSocket** | 实时双向同步、低延迟 |
| 多人在线游戏 | **WebSocket** | 最低延迟、二进制支持 |
| 简单状态轮询 | **短轮询** | 实现最简单、对服务器要求低 |
| 文件上传进度 | **SSE** 或 **短轮询** | 服务器→客户端的单向进度通知 |
| 物联网设备控制 | **WebSocket** | 双向控制指令、低开销 |
| 日志实时流 | **SSE** | 纯文本、单向推送 |

> 💡 **SSE vs WebSocket 选型总结**已移至 [[#17.5 SSE vs WebSocket 选型总结]]，紧接在 WebSocket 握手详解之后。

### 来源
- RFC 6455 — https://rfc-editor.org/rfc/rfc6455
- RFC 8441 — https://rfc-editor.org/rfc/rfc8441
- WHATWG HTML: SSE — https://html.spec.whatwg.org/server-sent-events.html
- MDN: SSE vs WebSocket — https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events

---

## 22. HTTP/3 与未来趋势

### HTTP/3 概述

HTTP/3 基于 QUIC 协议（运行在 UDP 之上），是 HTTP 协议的最新版本。它解决了 HTTP/2 中 TCP 层的队头阻塞问题。

> **注意**：关于 HTTP/3 的采用率数据，38.8% 通常指的是"网站覆盖率"（即支持 HTTP/3 的网站比例），而非"流量占比"。实际流量占比数据因来源和统计方法不同而有较大差异。

### HTTP 版本演进

| 版本 | 传输层 | 关键特性 |
|------|--------|---------|
| HTTP/1.0 | TCP | 短连接，每次请求新建连接 |
| HTTP/1.1 | TCP | 长连接、管道化（有队头阻塞） |
| HTTP/2 | TCP | 多路复用、头部压缩、服务器推送（TCP 层仍有队头阻塞） |
| HTTP/3 | QUIC/UDP | 彻底解决队头阻塞、0-RTT 连接建立、内置 TLS 1.3 |

### 来源
- RFC 9114: HTTP/3 — https://rfc-editor.org/rfc/rfc9114
- RFC 9000: QUIC — https://rfc-editor.org/rfc/rfc9000

---

## 23. 课程复盘

### 知识体系总结

本次学习构建了 HTTP 网络技术的完整知识体系：

```
HTTP 网络技术
├── 基础层
│   ├── 协议定义（无状态、请求-响应模型）
│   ├── 工作流程（DNS → TCP 握手 → 请求 → 响应 → 连接管理）
│   └── URL 结构（RFC 3986）
│
├── 语义层
│   ├── 方法（GET/POST/PUT/DELETE/PATCH）
│   ├── 状态码（1xx-5xx）
│   └── Header / Body
│
├── 实时通信层
│   ├── 短轮询（简单但低效）
│   ├── SSE（单向推送，HTTP 原生）
│   └── WebSocket（双向通信，低延迟）
│
└── 安全层
    ├── 同源策略（浏览器安全基石）
    ├── CORS（跨域资源共享）
    └── HTTPS（传输加密）
```

### 关键认知升级

1. **HTTP 无状态不是缺陷**：无状态设计使服务器可以水平扩展，状态管理交给应用层（Cookie/Session/Token）
2. **REST 不等于 CRUD**：REST 是架构风格，HTTP 方法的使用应遵循语义定义而非机械映射
3. **401 是"未认证"而非"未授权"**：虽然标准名为 Unauthorized，但语义上是 Unauthenticated
4. **CORS 不是服务器拦截**：请求已到达服务器并被处理，是浏览器根据响应头决定是否暴露给 JS
5. **SSE 连接限制的来源**：6 个连接限制并非来自 RFC 2616 的直接建议（RFC 2616 建议 2 个），而是浏览器厂商自行实现的惯例
6. **Safari 的 Max-Age 上限**：Safari/WebKit 硬编码上限为 600 秒（10 分钟），而非"无明确上限"
7. **PATCH 的幂等性是光谱**：RFC 5789 定义 PATCH 为非幂等，但允许特定实现使其幂等。实践中许多 PATCH 操作是幂等的
8. **HTTP/3 数据需区分**：38.8% 是"网站覆盖率"而非"流量占比"

### RFC 速查表

| 文档 | 编号 | 说明 |
|------|------|------|
| HTTP 语义 | RFC 9110 | 方法、状态码、头字段的完整定义 |
| HTTP/1.1 | RFC 9112 | 协议语法和连接管理 |
| HTTP/2 | RFC 9113 | HTTP/2 协议规范 |
| HTTP/3 | RFC 9114 | 基于 QUIC 的 HTTP/3 |
| URI 语法 | RFC 3986 | URL/URI 的标准语法 |
| PATCH 方法 | RFC 5789 | HTTP PATCH 方法规范 |
| WebSocket | RFC 6455 | WebSocket 协议规范 |
| WebSocket over HTTP/2 | RFC 8441 | HTTP/2 上的 WebSocket 引导 |
| Fetch 规范 | WHATWG | CORS 的权威定义 |
| HTML 规范 | WHATWG | SSE 的权威定义 |
| REST 论文 | Fielding 2000 | REST 架构风格原始定义 |

### 来源
- 本文基于 RFC 9110、RFC 5789、RFC 6455、RFC 3986、WHATWG Fetch Standard、WHATWG HTML Standard 等权威标准编写
- MDN Web Docs 提供了详尽的开发者文档
- Roy Fielding 的博士论文和博客是理解 REST 的第一手资料
