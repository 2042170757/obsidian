# Podman 与其他工具

## 概述
Podman 是 Docker 的主要替代方案，采用无守护进程架构，提供更好的安全性和与 Docker 的高度兼容性。

---

## 一、Podman 简介

Podman（Pod Manager）是一个开源的容器引擎，由 Red Hat 开发，用于开发、管理和运行 OCI 容器。

### 核心特性

| 特性 | 说明 |
|------|------|
| **无守护进程** | 不需要后台 daemon 进程，每个容器独立运行 |
| **Rootless** | 支持非 root 用户运行容器，更安全 |
| **兼容 Docker CLI** | 命令几乎完全兼容（`alias docker=podman`） |
| **Pod 支持** | 原生支持 Pod 概念（类似 K8s Pod） |
| **OCI 兼容** | 完全兼容 OCI 镜像和运行时标准 |

---

## 二、Podman vs Docker 对比

### 架构差异

```
Docker 架构：
┌──────────┐     REST API     ┌──────────────┐
│ docker   │ ──────────────►  │   dockerd    │
│ (CLI)    │                  │  (Daemon)    │
└──────────┘                  └──────┬───────┘
                                     │
                              ┌──────▼───────┐
                              │  containerd  │
                              └──────┬───────┘
                                     │
                              ┌──────▼───────┐
                              │     runc     │
                              └──────────────┘

Podman 架构：
┌──────────┐
│ podman   │ ──────────────►  直接调用 conmon/runc
│ (CLI)    │                  每个容器独立进程
└──────────┘                  无需守护进程
```

### 功能对比

| 维度 | Docker | Podman |
|------|--------|--------|
| **守护进程** | 需要 dockerd daemon | 无守护进程 |
| **Root 权限** | daemon 默认需要 root | 支持 rootless |
| **安全风险** | daemon 是单点攻击目标 | 无中心 daemon，攻击面更小 |
| **Docker Compose** | 原生支持 | 支持（`podman-compose` 或 `docker compose` 兼容） |
| **Docker Swarm** | 支持 | 不支持（推荐用 K8s） |
| **Pod 支持** | 不支持（需用 K8s） | 原生支持 |
| **镜像构建** | `docker build` | `podman build`（使用 Buildah） |
| **系统集成** | 独立安装 | 深度集成 systemd |
| **macOS/Windows** | Docker Desktop 原生支持 | 需要额外配置（Podman Machine） |

### 命令对比

| 操作 | Docker | Podman |
|------|--------|--------|
| 运行容器 | `docker run` | `podman run` |
| 查看容器 | `docker ps` | `podman ps` |
| 构建镜像 | `docker build` | `podman build` |
| 查看镜像 | `docker images` | `podman images` |
| 网络管理 | `docker network` | `podman network` |
| 卷管理 | `docker volume` | `podman volume` |
| 系统清理 | `docker system prune` | `podman system prune` |

> **兼容性提示：** 可以通过 `alias docker=podman` 将 Podman 作为 Docker 的直接替代。大多数 Docker 命令可以直接用 Podman 执行。

---

## 三、docker export/import vs docker load/save

这两组命令容易混淆，它们操作的对象不同：

### docker save / docker load（操作镜像）

```bash
# 保存镜像为 tar 文件（包含所有层和元数据）
docker save myapp:v1 > myapp-v1.tar
docker save -o myapp-v1.tar myapp:v1

# 从 tar 文件加载镜像
docker load < myapp-v1.tar
docker load -i myapp-v1.tar
```

| 特性 | 说明 |
|------|------|
| 操作对象 | 镜像（Image） |
| 保留内容 | 所有层、标签、元数据 |
| 用途 | 镜像迁移、离线分发 |

### docker export / docker import（操作容器）

```bash
# 导出容器的文件系统为 tar 文件
docker export mycontainer > mycontainer.tar
docker export -o mycontainer.tar mycontainer

# 从 tar 文件导入为镜像（丢失所有历史层和元数据）
docker import mycontainer.tar myapp:imported
cat mycontainer.tar | docker import - myapp:imported
```

| 特性 | 说明 |
|------|------|
| 操作对象 | 容器（Container）的文件系统快照 |
| 丢失内容 | 层历史、Dockerfile 指令、元数据 |
| 用途 | 容器快照、文件系统备份 |

### 对比总结

```
镜像（Image）                    容器（Container）
    │                                │
    ▼                                ▼
docker save ──► tar ──► docker load    docker export ──► tar ──► docker import
  (保留层)    (保留层)                  (扁平化)         (扁平化)
```

---

## 四、软链接概念（lrwxrwxrwx）

在 Linux 文件系统中，`lrwxrwxrwx` 表示这是一个**符号链接**（Symlink / Soft Link）。

### 文件权限解读

```
lrwxrwxrwx
│ │  │  │
│ │  │  └── 其他用户权限 (rwx)
│ │  └───── 组权限 (rwx)
│ └──────── 所有者权限 (rwx)
└────────── 文件类型 (l = link)
```

### 常见文件类型标识

| 标识 | 类型 | 示例 |
|------|------|------|
| `-` | 普通文件 | `-rw-r--r--` |
| `d` | 目录 | `drwxr-xr-x` |
| `l` | 符号链接 | `lrwxrwxrwx` |
| `c` | 字符设备 | `crw-rw----` |
| `b` | 块设备 | `brw-r-----` |

### 在 Docker 中的应用

```dockerfile
# 许多镜像中使用软链接来管理版本
# 例如 Python 镜像中：
# /usr/bin/python -> /usr/bin/python3.11
# /usr/bin/pip -> /usr/bin/pip3.11

# 在 docker build 中注意：
# COPY 会复制链接指向的文件（解引用）
# 如果需要保留链接本身，使用特定配置
```

---

## 五、0.0.0.0:8080 的含义

### IP 地址解析

| 地址 | 含义 | 监听范围 |
|------|------|----------|
| `0.0.0.0` | 所有网络接口 | 所有可用接口（包括 localhost、局域网、公网） |
| `127.0.0.1` | 本地回环地址 | 仅本机可访问 |
| `192.168.x.x` | 局域网地址 | 特定网段 |

### 在 Docker 中的应用

```bash
# 容器内服务监听 0.0.0.0:8080
# 意味着：接受来自任何网络接口的连接

# Dockerfile 中
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]

# 端口映射
docker run -p 8080:8080 myapp
# 宿主机 8080 → 容器内 0.0.0.0:8080
```

### 为什么容器内要用 0.0.0.0？

| 地址 | 在容器内监听 | 外部能否访问 |
|------|-------------|-------------|
| `0.0.0.0:8080` | 是 | **能**（通过端口映射） |
| `127.0.0.1:8080` | 是 | **不能**（只接受容器内部连接） |

> **关键点：** 在容器内，如果服务只监听 `127.0.0.1`，那么即使做了端口映射，外部也无法访问。必须监听 `0.0.0.0` 才能接受来自容器外部的连接。

---

## 相关概念
- [[wiki/Docker/Docker架构]]
- [[wiki/Docker/Docker镜像与分层]]
- [[wiki/Docker/容器生命周期与管理]]

## 来源
- Podman Official: https://podman.io/
- Docker Docs: https://docs.docker.com/engine/reference/commandline/save/
- Docker Docs: https://docs.docker.com/engine/reference/commandline/export/
