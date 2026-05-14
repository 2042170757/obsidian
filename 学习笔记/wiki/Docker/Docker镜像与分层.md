# Docker 镜像与分层

## 概述
Docker 镜像由多个只读层叠加而成，通过 Union File System 合并为统一视图，Copy-on-Write 机制确保高效存储和快速启动。

---

## 一、镜像的本质

Docker 镜像不是单个文件，而是一个**只读层的有序集合**。

```
┌──────────────────────────┐
│  Layer 4: CMD ["python"] │  ← 元数据层（不占空间）
├──────────────────────────┤
│  Layer 3: COPY . /app    │  ← 应用代码
├──────────────────────────┤
│  Layer 2: pip install    │  ← 依赖安装
├──────────────────────────┤
│  Layer 1: apt-get python │  ← 系统包
├──────────────────────────┤
│  Layer 0: Ubuntu 22.04   │  ← 基础镜像
└──────────────────────────┘
```

### 层的核心特性
| 特性 | 说明 |
|------|------|
| **不可变性（Immutable）** | 一旦创建，层的内容不可修改 |
| **可共享性** | 多个镜像/容器可以共享相同的层 |
| **内容寻址** | 层通过内容哈希标识，相同内容 = 相同层 |

---

## 二、Union File System（联合文件系统）

UnionFS 将多个目录（分支）挂载为一个统一的文件系统视图。

### Overlay2（当前默认存储驱动）

Docker 默认使用 Overlay2 驱动，基于 Linux 内核的 OverlayFS，由四个目录组成：

| 目录 | 权限 | 说明 |
|------|------|------|
| **LowerDir** | 只读 | 镜像层（可以有多个） |
| **UpperDir** | 可写 | 容器层（运行时变更写入此处） |
| **Merged** | 统一视图 | 容器进程实际看到的文件系统 |
| **WorkDir** | 内部 | OverlayFS 用于原子操作的工作目录 |

```
容器进程看到的文件系统 (Merged)
┌─────────────────────────────────┐
│                                 │
│  ┌─────────────────────────┐    │
│  │  UpperDir (可写)         │    │  ← 容器运行时的变更
│  ├─────────────────────────┤    │
│  │  LowerDir 3 (只读)      │    │  ← 镜像层
│  ├─────────────────────────┤    │
│  │  LowerDir 2 (只读)      │    │  ← 镜像层
│  ├─────────────────────────┤    │
│  │  LowerDir 1 (只读)      │    │  ← 基础镜像层
│  └─────────────────────────┘    │
└─────────────────────────────────┘
```

---

## 三、Copy-on-Write（CoW）机制

CoW 是一种延迟复制策略：**读时共享，写时复制**。

### 工作原理

| 操作 | CoW 行为 | 性能 |
|------|----------|------|
| **读取文件** | 直接从只读层读取，无需复制 | 极快 |
| **修改文件** | 先从只读层复制到可写层，再修改副本 | 首次较慢（需复制） |
| **删除文件** | 在可写层创建 "whiteout" 标记，隐藏只读层的文件 | 快 |

### CoW 操作步骤（以 overlay2 为例）

```
1. 文件搜索：容器请求修改 /etc/nginx/nginx.conf
2. 层级搜索：先在 UpperDir 搜索，未找到则搜索 LowerDir
3. 文件复制：找到后，从 LowerDir 复制到 UpperDir（copy-up）
4. 文件修改：对 UpperDir 中的副本进行修改
5. 后续访问：所有后续访问都指向 UpperDir 中的修改版本
```

### CoW 性能注意事项

> **WARNING：** 如果只读层有一个 1GB 的日志文件，你只追加 1 字节，Docker 必须先把整个 1GB 文件复制到容器层。因此，**写密集型数据（数据库、日志）应该使用 Volume，绕过 CoW 开销。**

---

## 四、只读模板与可写容器

| 概念 | 说明 |
|------|------|
| **镜像（Image）** | 只读模板，不可修改，可被多个容器共享 |
| **容器（Container）** | 在镜像之上添加一个薄的可写层（Container Layer） |

```
容器 A 的文件系统          容器 B 的文件系统
┌──────────────────┐     ┌──────────────────┐
│ Container Layer A│     │ Container Layer B│  ← 各自独立的可写层
├──────────────────┤     ├──────────────────┤
│                  │     │                  │
│   共享的镜像层    │     │   共享的镜像层    │  ← 只读，共享
│   (Read-Only)    │     │   (Read-Only)    │
│                  │     │                  │
└──────────────────┘     └──────────────────┘
```

**核心要点：**
- 容器停止后，可写层中的数据会丢失（除非使用 Volume）
- 多个容器可以同时运行同一个镜像，各自拥有独立的可写层
- 镜像层的共享使得存储效率极高

---

## 五、镜像命名规范

```
[仓库地址/] <用户名>/<镜像名>:<标签>
```

| 部分 | 是否必须 | 示例 | 说明 |
|------|----------|------|------|
| 仓库地址 | 否 | `harbor.example.com` | 默认为 Docker Hub |
| 用户名 | 否 | `library`（官方）/ `myuser` | Docker Hub 官方镜像省略 |
| 镜像名 | 是 | `nginx`, `python` | 镜像名称 |
| 标签 | 否 | `latest`, `1.25`, `v2.0` | 默认为 `latest` |

---

## 六、镜像操作命令

### 常用命令速查

| 命令 | 说明 | 示例 |
|------|------|------|
| `docker search <关键词>` | 在 Docker Hub 搜索镜像 | `docker search nginx` |
| `docker pull <镜像名>[:<标签>]` | 从 Registry 拉取镜像 | `docker pull nginx:1.25` |
| `docker images` / `docker image ls` | 列出本地镜像 | `docker images` |
| `docker rmi <镜像名或ID>` | 删除本地镜像 | `docker rmi nginx:1.25` |
| `docker image prune` | 删除所有悬空镜像 | `docker image prune` |
| `docker tag <源镜像> <新标签>` | 给镜像打标签 | `docker tag myapp myapp:v1` |
| `docker build -t <标签> <路径>` | 从 Dockerfile 构建镜像 | `docker build -t myapp .` |
| `docker push <镜像名>` | 推送镜像到 Registry | `docker push myuser/myapp:v1` |
| `docker history <镜像名>` | 查看镜像的层历史 | `docker history nginx` |
| `docker inspect <镜像名>` | 查看镜像详细信息 | `docker inspect nginx` |

---

## 相关概念
- [[wiki/Docker/Dockerfile]]
- [[wiki/Docker/Docker架构]]
- [[wiki/Docker/容器数据持久化]]

## 来源
- Docker Docs: https://docs.docker.com/storage/storagedriver/
- Docker Docs: https://docs.docker.com/get-started/docker-concepts/building-images/understanding-image-layers
- KodeKloud: https://kodekloud.com/blog/docker-image-layers/
- DEV Community: https://dev.to/sahillearninglinux/docker-filesystem-and-the-power-of-copy-on-write-cow-6a
