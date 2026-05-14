# Docker 架构

## 概述
Docker 采用 Client-Server 架构，由 Docker Client、Docker Daemon 和 Docker Registry 三大组件协同工作。

---

## 一、整体架构

```
┌─────────────────┐     REST API      ┌─────────────────┐
│  Docker Client  │ ───────────────►  │  Docker Daemon  │
│    (docker CLI) │  Unix Socket /    │    (dockerd)    │
└─────────────────┘  TCP Socket       └────────┬────────┘
                                               │
                    ┌──────────────────────────┼──────────────────────────┐
                    │                          │                          │
                    ▼                          ▼                          ▼
            ┌──────────────┐          ┌──────────────┐          ┌──────────────┐
            │   Images     │          │  Containers  │          │   Networks   │
            │   (镜像)      │          │   (容器)      │          │   (网络)      │
            └──────────────┘          └──────────────┘          └──────────────┘
                                               │
                                               ▼
                                       ┌──────────────┐
                                       │   Volumes    │
                                       │   (存储卷)    │
                                       └──────────────┘

                                       ┌──────────────┐
                                       │   Registry   │
                                       │  (镜像仓库)   │
                                       └──────────────┘
```

---

## 二、Docker Daemon（dockerd）

Docker Daemon 是运行在宿主机上的**后台常驻进程**，是 Docker 的"大脑"。

### 核心职责
| 职责 | 说明 |
|------|------|
| 监听 API 请求 | 通过 REST API 接收来自 Client 的指令 |
| 管理 Docker 对象 | 镜像、容器、网络、卷的生命周期管理 |
| 构建镜像 | 执行 `docker build`，解析 Dockerfile |
| 运行容器 | 创建和启动容器进程 |
| 多主机通信 | 可与其他 Daemon 通信（如 Swarm 集群） |

### 底层组件链
```
dockerd → containerd → containerd-shim → runc
```

| 组件 | 说明 |
|------|------|
| **dockerd** | Docker 守护进程，对外暴露 REST API |
| **containerd** | 容器运行时，管理容器生命周期 |
| **containerd-shim** | 每个容器一个 shim 进程，负责容器的 stdin/stdout |
| **runc** | 底层 OCI 运行时，真正 fork 并隔离容器进程 |

### 数据存储
- Linux 默认路径：`/var/lib/docker`
- 较新版本可能使用：`/var/lib/containerd`

---

## 三、Docker Client（docker）

Docker Client 是用户与 Docker 交互的**命令行工具**。

### 工作原理
1. 用户输入命令（如 `docker run nginx`）
2. Client 将命令转换为 REST API 请求
3. 通过 Unix Socket（默认）或 TCP Socket 发送给 Daemon
4. Daemon 执行操作并返回结果

### 通信方式
| 方式 | 路径/格式 | 说明 |
|------|-----------|------|
| Unix Socket（默认） | `/var/run/docker.sock` | 本机通信，性能最佳 |
| TCP Socket | `tcp://host:port` | 远程通信，需配置 TLS |
| SSH | `ssh://user@host` | 安全远程通信 |

> **核心理解：** Client 是无状态的轻量级工具，它不做任何实际工作，只是"传话员"。真正的重活由 Daemon 完成。

---

## 四、Docker Registry（镜像仓库）

Registry 是存储和分发 Docker 镜像的服务。

### 类型
| 类型 | 示例 | 说明 |
|------|------|------|
| **公共仓库** | Docker Hub | 默认仓库，包含大量官方和社区镜像 |
| **私有仓库** | Harbor, AWS ECR, Google Artifact Registry | 企业内部使用，更安全可控 |

### 核心操作
| 操作 | 命令 | 方向 |
|------|------|------|
| **拉取** | `docker pull nginx` | Registry → 本地 |
| **推送** | `docker push myapp:v1` | 本地 → Registry |

### 镜像命名规范
```
[仓库地址/]<用户名>/<镜像名>:<标签>
```

示例：
| 镜像名称 | 解析 |
|----------|------|
| `nginx` | Docker Hub 官方镜像，标签默认 `latest` |
| `nginx:1.25` | Docker Hub 官方镜像，标签 `1.25` |
| `myuser/myapp:v1` | Docker Hub 用户镜像 |
| `harbor.example.com/prod/myapp:v1` | 私有仓库镜像 |

---

## 五、核心管理对象

Docker Engine 管理四类核心资源：

| 对象 | 说明 | 命令前缀 |
|------|------|----------|
| **Images（镜像）** | 只读模板，由分层文件系统快照和元数据组成 | `docker image` |
| **Containers（容器）** | 镜像的运行实例，包含可写层和运行中的进程 | `docker container` |
| **Networks（网络）** | 虚拟网络，实现容器间及容器与外部的通信 | `docker network` |
| **Volumes（存储卷）** | 持久化存储，独立于容器生命周期 | `docker volume` |

---

## 六、完整工作流程

当执行 `docker run nginx` 时：

```
1. Client → Daemon:  发送 REST API 请求 "创建并运行 nginx 容器"
2. Daemon:           检查本地是否存在 nginx:latest 镜像
3. Daemon → Registry: 如果本地没有，从 Docker Hub 拉取镜像
4. Daemon:           使用镜像创建容器（添加可写层）
5. Daemon:           通过 containerd → runc 启动容器进程
6. Client:           收到容器 ID，显示输出
```

---

## 相关概念
- [[wiki/Docker/容器与虚拟机]]
- [[wiki/Docker/Docker镜像与分层]]
- [[wiki/Docker/Dockerfile]]

## 来源
- KodeKloud: https://kodekloud.com/blog/docker-architecture/
- GeeksforGeeks: https://www.geeksforgeeks.org/architecture-of-docker
- Spacelift: https://spacelift.dev/blog/docker-architecture
- Docker Docs: https://docs.docker.com/engine/
