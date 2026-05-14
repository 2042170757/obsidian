# Docker Compose

## 概述
Docker Compose 是一个用于定义和运行多容器 Docker 应用的工具。通过一个 YAML 文件配置所有服务，一条命令启动整个应用栈。

---

## 一、为什么需要 Compose？

| 场景 | 没有 Compose | 有 Compose |
|------|-------------|-----------|
| 启动多容器应用 | 逐个 `docker run`，手动配置网络和卷 | 一条 `docker compose up` |
| 服务依赖 | 手动确保启动顺序 | 自动处理依赖和健康检查 |
| 环境配置 | 命令行参数冗长 | YAML 文件清晰声明 |
| 团队协作 | "请运行这一长串命令" | "请运行 `docker compose up`" |

---

## 二、compose.yml 结构

### 基本结构

```yaml
# 顶层结构
services:      # 服务定义（必需）
  web:
    ...
  db:
    ...

networks:      # 网络定义（可选）
  frontend:
    ...

volumes:       # 卷定义（可选）
  db_data:
    ...
```

### 完整示例

```yaml
services:
  # Web 应用服务
  web:
    build: ./web                # 从 Dockerfile 构建
    ports:
      - "8080:3000"            # 端口映射
    environment:
      - DATABASE_URL=postgresql://postgres:secret@db:5432/myapp
    depends_on:
      db:
        condition: service_healthy  # 等待 db 健康后再启动
    networks:
      - frontend
      - backend
    restart: unless-stopped

  # 数据库服务
  db:
    image: postgres:16-alpine   # 使用现成镜像
    volumes:
      - postgres_data:/var/lib/postgresql/data  # 持久化数据
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}  # 从环境变量读取
      POSTGRES_DB: myapp
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    networks:
      - backend
    restart: unless-stopped

  # Redis 缓存
  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  postgres_data:
```

---

## 三、健康检查（Health Check）

### 在 compose.yml 中定义

```yaml
services:
  web:
    image: myapp:latest
    ports:
      - "8080:3000"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s       # 检查间隔
      timeout: 10s        # 超时时间
      retries: 3          # 失败重试次数
      start_period: 10s   # 启动宽限期
```

### 健康检查参数说明

| 参数 | 说明 | 建议值 |
|------|------|--------|
| `test` | 检查命令 | `["CMD", "curl", "-f", "http://localhost/health"]` |
| `interval` | 检查频率 | 10-30s |
| `timeout` | 单次检查超时 | 5-10s |
| `retries` | 连续失败次数后标记为 unhealthy | 3-5 |
| `start_period` | 启动宽限期（此期间失败不计入） | 根据应用启动时间设置 |

### CMD vs CMD-SHELL

```yaml
# CMD：直接执行，不经过 shell
test: ["CMD", "curl", "-f", "http://localhost:3000/health"]

# CMD-SHELL：通过 shell 执行，支持变量展开和管道
test: ["CMD-SHELL", "curl -f http://localhost:${PORT}/health || exit 1"]
```

### 常见服务的健康检查模板

```yaml
# PostgreSQL
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U postgres"]
  interval: 10s
  timeout: 5s
  retries: 5

# Redis
healthcheck:
  test: ["CMD", "redis-cli", "ping"]
  interval: 10s
  timeout: 5s
  retries: 3

# MySQL
healthcheck:
  test: ["CMD-SHELL", "mysqladmin ping -h localhost"]
  interval: 10s
  timeout: 5s
  retries: 5

# HTTP 服务
healthcheck:
  test: ["CMD-SHELL", "curl -f http://localhost:8080/health || exit 1"]
  interval: 30s
  timeout: 5s
  retries: 3
  start_period: 15s
```

### 健康状态与服务依赖

```yaml
services:
  app:
    image: myapp:latest
    depends_on:
      db:
        condition: service_healthy              # 等待 db 健康
      redis:
        condition: service_healthy              # 等待 redis 健康
      migrations:
        condition: service_completed_successfully  # 等待迁移完成

  db:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      ...

  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      ...

  migrations:
    image: myapp:latest
    command: python manage.py migrate
    depends_on:
      db:
        condition: service_healthy
```

### Dockerfile vs Compose 中的健康检查

| 位置 | 适用场景 |
|------|----------|
| **Dockerfile** | 默认健康检查，随镜像分发，所有人都能用 |
| **compose.yml** | 环境特定检查，可覆盖 Dockerfile 中的定义 |

> **注意：** Compose 中的 healthcheck 会**完全覆盖** Dockerfile 中的 HEALTHCHECK，而不是合并。

---

## 四、自动 DNS 解析

Docker Compose 自动为每个服务创建 DNS 记录，服务之间可以通过**服务名**直接通信。

```yaml
services:
  web:
    image: myapp
    # 在 web 容器中可以直接用 "db" 访问数据库服务
    environment:
      DATABASE_URL: postgresql://postgres:secret@db:5432/myapp

  db:
    image: postgres:16
```

- 服务名就是主机名
- 同一 Compose 文件中的所有服务默认在同一网络中
- 无需手动配置 `--link` 或 IP 地址

---

## 五、CLI 命令

### 常用命令

| 命令 | 说明 |
|------|------|
| `docker compose up` | 创建并启动所有服务 |
| `docker compose up -d` | 后台启动 |
| `docker compose down` | 停止并删除容器、网络 |
| `docker compose down -v` | 同时删除卷 |
| `docker compose ps` | 查看服务状态 |
| `docker compose logs` | 查看所有服务日志 |
| `docker compose logs -f` | 实时跟踪日志 |
| `docker compose logs -f web` | 跟踪指定服务日志 |
| `docker compose exec web bash` | 进入运行中的服务容器 |
| `docker compose build` | 构建/重新构建服务镜像 |
| `docker compose pull` | 拉取服务镜像 |
| `docker compose restart` | 重启所有服务 |
| `docker compose stop` | 停止服务（不删除） |
| `docker compose start` | 启动已停止的服务 |
| `docker compose config` | 验证并查看解析后的配置 |
| `docker compose top` | 查看各服务中运行的进程 |

### 常用组合

```bash
# 重新构建并启动（代码变更后）
docker compose up -d --build

# 只重启某个服务
docker compose restart web

# 查看资源使用
docker compose top

# 清理一切（包括卷）
docker compose down -v --rmi all
```

---

## 六、环境变量管理

### 方式 1：.env 文件（推荐）

```bash
# .env 文件（放在 compose.yml 同级目录）
DB_PASSWORD=supersecret
DB_VERSION=16
APP_PORT=8080
```

```yaml
# compose.yml 中引用
services:
  db:
    image: postgres:${DB_VERSION}
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
  app:
    ports:
      - "${APP_PORT}:3000"
```

### 方式 2：environment 直接定义

```yaml
services:
  db:
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
```

### 方式 3：env_file 引用

```yaml
services:
  app:
    env_file:
      - .env
      - .env.production
```

### 环境变量优先级

```
compose.yml environment > .env 文件 > 宿主机环境变量
```

### 注意事项

```yaml
# 健康检查中使用环境变量需要用 CMD-SHELL
# 错误写法：
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:${PORT}/health"]

# 正确写法：
healthcheck:
  test: ["CMD-SHELL", "curl -f http://localhost:${PORT}/health || exit 1"]
```

---

## 相关概念
- [[wiki/Docker/Docker网络]]
- [[wiki/Docker/容器生命周期与管理]]
- [[wiki/Docker/Dockerfile]]

## 来源
- Docker Docs: https://docs.docker.com/compose/
- Docker Docs: https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation
- Docker Recipes: https://docker.recipes/docs/healthchecks-dependencies
- JustAnotherUptime: https://blog.justanotheruptime.com/posts/2025_06_24_docker_compose_healthcheck/
