# Dockerfile

## 概述
Dockerfile 是一个文本文件，包含一系列指令，用于自动化构建 Docker 镜像。每条指令创建一个新的镜像层。

---

## 一、基本语法和指令

### 核心指令速查表

| 指令 | 说明 | 示例 |
|------|------|------|
| `FROM` | 指定基础镜像（必须是第一条非注释指令） | `FROM python:3.11-slim` |
| `RUN` | 在构建时执行命令（安装依赖等） | `RUN apt-get update && apt-get install -y curl` |
| `COPY` | 从构建上下文复制文件到镜像 | `COPY . /app` |
| `ADD` | 类似 COPY，但支持 URL 和自动解压 tar | `ADD https://example.com/file.tar.gz /app/` |
| `WORKDIR` | 设置工作目录 | `WORKDIR /app` |
| `ENV` | 设置环境变量 | `ENV NODE_ENV=production` |
| `EXPOSE` | 声明容器监听的端口（文档性质） | `EXPOSE 8080` |
| `CMD` | 设置容器启动时的默认命令（可被覆盖） | `CMD ["python", "app.py"]` |
| `ENTRYPOINT` | 设置容器的主入口点（不易被覆盖） | `ENTRYPOINT ["python"]` |
| `USER` | 设置后续指令的运行用户 | `USER appuser` |
| `VOLUME` | 创建挂载点 | `VOLUME /data` |
| `ARG` | 定义构建时的变量 | `ARG VERSION=1.0` |
| `LABEL` | 添加元数据 | `LABEL maintainer="dev@example.com"` |
| `HEALTHCHECK` | 定义健康检查 | `HEALTHCHECK CMD curl -f http://localhost/` |

### CMD vs ENTRYPOINT 的区别

```
ENTRYPOINT ["python"]
CMD ["app.py"]

# 结果：python app.py
# 覆盖：docker run myapp script.py → python script.py
# CMD 提供默认参数，ENTRYPOINT 设置主命令
```

| 特性 | CMD | ENTRYPOINT |
|------|-----|------------|
| 用途 | 默认命令/参数 | 主入口命令 |
| 被 `docker run` 参数覆盖 | 是 | 否（参数追加） |
| 每个 Dockerfile 数量 | 只有最后一个生效 | 只有最后一个生效 |

### RUN 的两种形式

```dockerfile
# Shell 形式（通过 /bin/sh -c 执行）
RUN apt-get update && apt-get install -y curl

# Exec 形式（直接执行，不经过 shell）
RUN ["apt-get", "install", "-y", "curl"]
```

---

## 二、Python Web 应用构建示例

### 基础版本

```dockerfile
# syntax=docker/dockerfile:1

# 1. 指定基础镜像
FROM python:3.11-slim

# 2. 设置环境变量
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# 3. 设置工作目录
WORKDIR /app

# 4. 先复制依赖文件（利用缓存）
COPY requirements.txt .

# 5. 安装依赖
RUN pip install --no-cache-dir -r requirements.txt

# 6. 复制应用代码
COPY . .

# 7. 创建非 root 用户
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# 8. 声明端口
EXPOSE 8000

# 9. 启动命令
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 构建和运行

```bash
# 构建镜像
docker build -t my-python-app .

# 运行容器
docker run -d -p 8080:8000 --name myapp my-python-app
```

---

## 三、多阶段构建（Multi-stage Build）

### 为什么需要多阶段构建？

| 问题 | 单阶段构建 | 多阶段构建 |
|------|-----------|-----------|
| 镜像体积 | 包含编译器、开发工具等 | 只包含运行时必需文件 |
| 安全性 | 攻击面大（有不必要的工具） | 攻击面小（最小化镜像） |

### 工作原理

```
Stage 1 (Builder)          Stage 2 (Runtime)
┌─────────────────┐       ┌─────────────────┐
│ 完整开发镜像     │       │ 精简运行时镜像   │
│ - 编译器        │       │ - 运行时环境     │
│ - 构建工具      │ ───►  │ - 编译产物       │
│ - 开发依赖      │ COPY  │ - 生产依赖       │
│ - 源代码        │ --from│                  │
└─────────────────┘       └─────────────────┘
  构建完成后丢弃            最终镜像
```

### Python 多阶段构建示例

```dockerfile
# ────── Stage 1: 构建依赖 (Builder Stage) ──────
FROM python:3.11-slim AS builder

WORKDIR /app

# 创建虚拟环境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# ────── Stage 2: 运行时 (Runtime Stage) ──────
FROM python:3.11-slim

WORKDIR /app

# 从 builder 阶段复制虚拟环境
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 复制应用代码
COPY . .

# 创建非 root 用户
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Go 多阶段构建示例

```dockerfile
# ────── Stage 1: 编译 ──────
FROM golang:1.21 AS build
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server .

# ────── Stage 2: 运行 ──────
FROM alpine:3.19
RUN apk --no-cache add ca-certificates
WORKDIR /app
COPY --from=build /app/server /app/
EXPOSE 8080
CMD ["/app/server"]
```

### 多阶段构建的高级用法

```dockerfile
# 命名阶段，便于引用
FROM python:3.11-slim AS base
RUN pip install requests

# 多个构建目标
FROM base AS build1
COPY script1.py .
RUN python script1.py

FROM base AS build2
COPY script2.py .
RUN python script2.py

# 只构建特定阶段
# docker build --target build1 -t debug-app .
```

---

## 四、docker build 和 docker tag

### docker build 命令

```bash
# 基本构建
docker build -t myapp:latest .

# 指定 Dockerfile 路径
docker build -f Dockerfile.prod -t myapp:prod .

# 使用构建参数
docker build --build-arg VERSION=2.0 -t myapp:v2 .

# 多平台构建
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .
```

### docker tag 命令

```bash
# 语法：docker tag <源镜像>:<源标签> <目标镜像>:<目标标签>
docker tag myapp:latest myuser/myapp:v1.0
docker tag myapp:latest harbor.example.com/prod/myapp:v1.0
```

---

## 五、构建上下文优化

### 什么是构建上下文？

执行 `docker build .` 时，`.` 指定的目录就是**构建上下文**。Docker 会将整个目录发送给 Daemon。

### .dockerignore 文件

```dockerignore
# 版本控制
.git
.gitignore

# 依赖目录
node_modules
__pycache__
*.pyc

# IDE 配置
.vscode
.idea

# 文档
README.md
docs/

# 敏感文件
.env
*.key
*.pem
```

### 优化技巧

| 技巧 | 说明 |
|------|------|
| **使用 .dockerignore** | 排除不需要的文件，减小上下文体积 |
| **先复制依赖文件** | `COPY requirements.txt .` 在 `COPY . .` 之前，利用构建缓存 |
| **合并 RUN 指令** | 减少镜像层数 |
| **清理缓存** | `rm -rf /var/lib/apt/lists/*` |
| **使用 slim/alpine 基础镜像** | `python:3.11-slim`（45MB） vs `python:3.11`（125MB） |
| **使用多阶段构建** | 分离构建环境和运行环境 |

---

## 相关概念
- [[wiki/Docker/Docker镜像与分层]]
- [[wiki/Docker/容器生命周期与管理]]

## 来源
- Docker Docs: https://docs.docker.com/reference/builder/
- Docker Docs: https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile/
- Docker Docs: https://docs.docker.com/get-started/docker-concepts/building-images/multi-stage-builds
- Docker Docs: https://docs.docker.com/articles/dockerfile_best-practices
