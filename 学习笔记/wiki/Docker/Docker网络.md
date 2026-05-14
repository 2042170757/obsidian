# Docker 网络

## 概述
Docker 提供多种网络驱动来实现容器间及容器与外部的通信。最常用的是 bridge（桥接）、host（宿主机）和 none（无网络）。

---

## 一、网络驱动概览

| 驱动 | 说明 | 使用场景 |
|------|------|----------|
| **bridge** | 默认驱动，创建虚拟网桥 | 单机多容器通信 |
| **host** | 共享宿主机网络栈 | 需要最高网络性能 |
| **none** | 完全禁用网络 | 安全隔离场景 |
| **overlay** | 跨主机通信 | Docker Swarm 集群 |
| **macvlan** | 容器拥有独立 MAC 地址 | 需要容器在局域网中可见 |
| **ipvlan** | 连接到外部 VLAN | 企业网络集成 |

---

## 二、bridge（默认桥接网络）

### 默认 bridge vs 自定义 bridge

| 特性 | 默认 bridge | 自定义 bridge |
|------|-------------|---------------|
| DNS 解析 | 不支持（只能用 IP） | 支持（容器名自动解析） |
| 自动隔离 | 否（所有容器共享） | 是（项目级隔离） |
| 运行时连接/断开 | 不支持 | 支持 |
| 推荐用于生产 | 否 | **是** |

> **最佳实践：** 始终使用自定义 bridge 网络。默认 bridge 不支持容器间 DNS 解析。

### 创建和使用自定义网络

```bash
# 创建自定义桥接网络
docker network create mynet

# 在指定网络中运行容器
docker run -d --name web --network mynet nginx
docker run -d --name db --network mynet postgres:16

# 在 web 容器中可以直接通过容器名访问 db
docker exec web ping db  # 自动 DNS 解析！

# 查看网络
docker network ls

# 查看网络详情
docker network inspect mynet

# 将运行中的容器连接到网络
docker network connect mynet my-container

# 断开网络
docker network disconnect mynet my-container

# 删除网络
docker network rm mynet
```

### 网络架构

```
┌─────────────────────────────────────────────┐
│                宿主机                        │
│                                             │
│  ┌─────────┐    ┌──────────┐    ┌─────────┐│
│  │ Container│    │  Bridge  │    │Container││
│  │   web    │────│  (mynet) │────│   db    ││
│  │ 172.18.0.2│   │ 172.18.0.1│  │172.18.0.3││
│  └─────────┘    └────┬─────┘    └─────────┘│
│                      │                      │
│                      │ NAT                   │
│                      ▼                      │
│                ┌──────────┐                 │
│                │  eth0    │                 │
│                │ (宿主机)  │                 │
│                └──────────┘                 │
└─────────────────────────────────────────────┘
```

---

## 三、host（宿主机网络）

容器直接使用宿主机的网络栈，**没有网络隔离**。

```bash
docker run -d --network host nginx
# nginx 直接绑定宿主机的 80 端口，无需 -p 端口映射
```

| 优点 | 缺点 |
|------|------|
| 最佳网络性能（无 NAT 开销） | 没有网络隔离 |
| 无需端口映射 | 端口冲突风险 |
| | 仅 Linux 支持（macOS/Windows 不支持） |

**适用场景：** 对网络性能要求极高的场景，如 DNS 服务（Pi-hole）、网络监控工具。

---

## 四、none（无网络）

完全禁用容器的网络功能。

```bash
docker run -d --network none alpine
# 容器只有 loopback 接口，无法访问外部网络
```

**适用场景：** 安全敏感的计算任务，不需要网络访问。

---

## 五、自定义网络和 DNS 解析

### DNS 解析机制

| 网络类型 | DNS 解析方式 |
|----------|-------------|
| 默认 bridge | 不支持自动 DNS，需使用 `--link`（已废弃）或手动配置 |
| 自定义网络 | Docker 内置 DNS 服务器（`127.0.0.11`），自动解析容器名 |
| host | 使用宿主机 DNS |

### DNS 配置

```bash
# 自定义 DNS 服务器
docker run -d --dns 8.8.8.8 --dns 8.8.4.4 nginx

# 设置 DNS 搜索域
docker run -d --dns-search example.com nginx

# 设置主机名
docker run -d --hostname myapp nginx
```

### Docker Compose 中的网络

```yaml
services:
  proxy:
    image: nginx
    networks:
      - frontend
  app:
    image: myapp
    networks:
      - frontend
      - backend
  db:
    image: postgres:16
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

在上面的配置中：
- `proxy` 只能访问 `app`（通过 frontend 网络）
- `db` 只能被 `app` 访问（通过 backend 网络）
- `app` 可以同时访问 `proxy` 和 `db`
- 实现了**网络隔离**：前端无法直接访问数据库

---

## 六、端口映射回顾

```bash
# -p HOST:CONTAINER
docker run -d -p 8080:80 nginx
# 宿主机 8080 → 容器 80

# 0.0.0.0 的含义：
# -p 8080:80       等同于 -p 0.0.0.0:8080:80
# 监听所有网络接口（允许外部访问）
# -p 127.0.0.1:8080:80  只监听 localhost（仅本机访问）
```

| 地址 | 含义 | 安全性 |
|------|------|--------|
| `0.0.0.0:8080` | 监听所有接口 | 较低（外部可访问） |
| `127.0.0.1:8080` | 只监听 localhost | 高（仅本机） |
| `192.168.1.100:8080` | 监听指定 IP | 中（指定网络） |

---

## 相关概念
- [[wiki/Docker/Docker架构]]
- [[wiki/Docker/Docker Compose]]
- [[wiki/Docker/容器生命周期与管理]]

## 来源
- Docker Docs: https://docs.docker.com/engine/network/
- Docker Docs: https://docs.docker.com/engine/network/tutorials/standalone/
- Docker Docs: https://docs.docker.com/engine/network/drivers
- Self Host Setup: https://selfhostsetup.com/posts/docker-networking-bridge-host-macvlan/
