# 别动我的容器！Docker修仙传

> [!quote]
> "在我机器上能跑啊！" —— 程序员三大经典谎言之首

---

## 第一章 容器解决了什么问题？

### 核心心法：环境一致性

容器修炼的核心心法只有一个：**让"在我机器上能跑"这句话变成真话。**

在容器出现之前，部署软件就像在不同的世界之间搬运仙丹——你在炼丹炉（开发环境）里炼好的丹药，到了别的宗门（生产环境）就可能水土不服、药效全无。

> [!tip] 比喻：标准化集装箱
> 想象国际贸易中的集装箱。在集装箱出现之前，货物的装卸、转运方式五花八门，效率极低。集装箱的出现统一了货物的包装标准——无论货物是什么，都装进同样规格的箱子里，可以用同样的吊车、同样的卡车、同样的轮船来运输。
>
> Docker容器就像软件世界的"集装箱"：把应用、依赖、配置打包成一个标准化的单元，无论在哪里运行，行为都是一致的。

### 误区澄清

| 误区                         | 真相                                                                                 |
| -------------------------- | ---------------------------------------------------------------------------------- |
| "Build once, run anywhere" | 实际是 "Package once, run anywhere **on the same architecture**"。x86上构建的镜像无法直接在ARM上运行 |
| "秒级启动"                     | 需要区分：镜像已缓存时确实很快（100-500ms）；首次拉取大镜像可能需要数分钟                                          |
| "容器能解决所有部署问题"              | 容器不能完全解决配置管理、密钥管理、数据库schema迁移等问题                                                   |
| "容器总是比VM更好"                | 对简单单体应用，容器可能引入不必要的复杂性（过度工程化）                                                       |

---

## 第二章 容器 vs 虚拟机：独栋别墅与公寓楼之争

### 架构对比

```
        虚拟机 (VM)                          容器 (Container)
┌─────────────────────────┐        ┌─────────────────────────┐
│     Application A       │        │  App A  │  App B  │ App C│
├─────────────────────────┤        ├─────────┴─────────┴──────┤
│     Guest OS            │        │   Bins/Libs (共享)        │
├─────────────────────────┤        ├──────────────────────────┤
│     Virtual Hardware    │        │   Container Runtime      │
├─────────────────────────┤        │   (containerd / runc)    │
│     Hypervisor          │        ├──────────────────────────┤
├─────────────────────────┤        │   Host OS Kernel (共享)   │
│     Host OS             │        ├──────────────────────────┤
├─────────────────────────┤        │   Host OS                │
│     Physical Hardware   │        ├──────────────────────────┤
└─────────────────────────┘        │   Physical Hardware      │
                                   └──────────────────────────┘
```

> [!tip] 比喻：独栋别墅 vs 公寓楼
> **虚拟机 like 独栋别墅**：每栋别墅都有自己的地基（Guest OS）、自己的水电系统（虚拟硬件）、自己的门禁（Hypervisor隔离）。安全性和独立性很强，但每栋都要占用土地（资源），建造成本高（启动慢）。
>
> **容器 like 公寓楼**：所有住户共享同一栋楼的地基（宿主机内核）和主体结构，但每户有自己的独立空间（Namespace隔离）。建造成本低（启动快）、空间利用率高（资源效率高），但如果楼体结构（内核）出问题，所有住户都受影响。

### 关键差异量化

| 维度 | 虚拟机 | 容器 |
|------|--------|------|
| **磁盘占用** | 2-50 GB | 5 MB - 1.5 GB |
| **启动时间** | 30秒 - 5分钟 | 100-500毫秒 |
| **内存开销** | 512 MB - 2 GB | 1-5 MB |
| **CPU 开销** | 5-15% | < 1-2% |
| **隔离级别** | 硬件级（非常强） | 操作系统级（较强） |

### 争议性问题：容器安全性 vs 虚拟机安全性

| 安全维度 | 容器 | 虚拟机 |
|----------|------|--------|
| 内核攻击面 | 共享内核，内核漏洞可能影响所有容器 | 独立内核，攻击面隔离 |
| 逃逸难度 | 相对较低（Namespace弱点、系统调用暴露） | 非常高（需突破Hypervisor） |
| CVE影响范围 | 内核漏洞影响所有容器 | 仅影响单个VM |
| 安全增强方案 | gVisor、Kata Containers（有性能开销） | 天然强隔离 |

> [!warning] 结论
> 对于多租户或不可信工作负载，VM提供更强的安全边界。现代最佳实践是"VM内跑容器"——用VM提供基础设施层的强隔离，VM内部运行Docker容器实现应用部署的密度和一致性。

---

## 第三章 Namespace 与 Cgroup：公寓的房间隔断与水电表

### Namespace（命名空间）—— 隔离机制

> [!tip] 比喻：公寓的房间隔断
> 想象一栋公寓楼。Namespace就像房间的隔断墙：
> - **PID Namespace**：每户人家只能看到自己家的人，看不到邻居家有谁
> - **Network Namespace**：每户有自己的门牌号和电话线，互不干扰
> - **Mount Namespace**：每户有自己的储物空间，看不到邻居家的东西
> - **UTS Namespace**：每户可以取自己的户名，不影响别人
> - **IPC Namespace**：每户有自己的通讯系统，消息不外泄
> - **User Namespace**：每户有自己的户主权限，互不干涉

### Cgroup（控制组）—— 资源限制

> [!tip] 比喻：公寓的水电表
> Cgroup就像公寓每户的独立水电表：
> - **CPU Cgroup**：限制每户的用电功率（CPU使用率），防止一户把整栋楼的电都用完
> - **Memory Cgroup**：限制每户的用水量（内存使用），超限就会被"断水"（OOM Kill）
> - **I/O Cgroup**：限制每户的煤气流量（磁盘I/O），避免一户独占资源
> - **Network Cgroup**：限制每户的网络带宽

### 争议性问题：容器的隔离强度

**需要关注的安全风险：**
- **内核漏洞逃逸**：攻击者利用内核漏洞突破Namespace隔离
- **系统调用暴露**：容器共享内核意味着暴露了大量系统调用接口
- **Namespace弱点**：某些Namespace配置不当可能导致信息泄露

**增强方案：**

| 方案 | 原理 | 权衡 |
|------|------|------|
| gVisor | 用户态内核，拦截系统调用 | 性能损失，兼容性限制 |
| Kata Containers | 轻量级VM，每个容器一个独立内核 | 资源开销增加 |
| Firecracker | 极简VMM，专为serverless设计 | AWS主导，生态较小 |

---

## 第四章 Docker 架构：餐厅的分工体系

### 整体架构

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
            └──────────────┘          └──────────────┘          └──────────────┘
```

### 底层组件链

```
dockerd → containerd → containerd-shim → runc
```

> [!tip] 比喻：餐厅的分工体系
> - **dockerd（餐厅经理）**：接收顾客（用户）的点单，统筹全局
> - **containerd（厨师长）**：管理厨房的日常运作，安排做菜
> - **containerd-shim（服务员）**：每个包间配一个，负责传菜和收碟
> - **runc（厨师）**：真正动手做菜的人，负责具体的烹饪工作

### 争议性问题：Docker Daemon 的安全风险

**Docker Daemon 的安全问题：**
- Daemon 默认以 root 权限运行
- 挂载 `docker.sock` 的容器可以获得宿主机 root 权限
- 单点攻击目标，一旦被攻破，整个宿主机沦陷

**Podman 的优势：**
- 无守护进程架构，每个容器独立进程
- 原生支持 rootless 模式
- 攻击面更小

---

## 第五章 镜像与分层：透明文件夹的魔法

### 镜像的本质

Docker 镜像不是单个文件，而是一个**只读层的有序集合**。

```
┌──────────────────────────┐
│  Layer 4: CMD ["python"] │  ← 元数据层
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

> [!tip] 比喻：透明文件夹叠加
> 想象你有一叠透明文件夹，每个文件夹里有不同颜色的纸片。当你把它们叠在一起看时，你会看到所有纸片叠加的效果。如果多个文件夹在同一位置都有纸片，你只能看到最上面那个。
>
> Overlay2的工作原理类似：
> - **LowerDir（底层文件夹）**：只读的镜像层，像透明文件夹一样叠加
> - **UpperDir（上层文件夹）**：可写的容器层，所有修改都记录在这里
> - **Merged（合并视图）**：你最终看到的叠加效果
> - **WorkDir（工作目录）**：OverlayFS内部使用的临时空间

### Copy-on-Write（CoW）机制

> [!tip] 比喻：图书馆的借阅系统
> 想象一个图书馆：
> - **书籍原件（只读层）**：图书馆的藏书，所有人都可以阅读，但不能在上面写字
> - **借阅副本（可写层）**：如果你想在书上做笔记，图书馆会给你复印一份，你在副本上写
> - **后续借阅**：下次你再借这本书，图书馆会给你上次的副本，而不是原件
>
> CoW的工作方式类似：
> - 读取文件：直接从只读层读取（借阅原件）
> - 修改文件：先复制到可写层，再修改（复印后做笔记）
> - 删除文件：在可写层标记"已删除"（在目录上盖"已借出"章）

> [!warning] CoW 的性能陷阱
> 如果只读层有一个 1GB 的日志文件，你只追加 1 字节，Docker 必须先把整个 1GB 文件复制到容器层。因此，**写密集型数据（数据库、日志）应该使用 Volume，绕过 CoW 开销。**

---

## 第六章 Dockerfile：建筑图纸与多阶段施工

### Dockerfile 是什么？

> [!tip] 比喻：建筑图纸（或菜谱）
> Dockerfile就像建筑图纸：
> - **FROM**：选择地基类型（用什么基础镜像）
> - **RUN**：施工步骤（安装依赖、配置环境）
> - **COPY**：搬运建材（复制文件到镜像）
> - **WORKDIR**：设定工作区域（设置工作目录）
> - **CMD**：入住规则（容器启动时执行的命令）
>
> 每一条指令就是一层施工，最终建成一栋完整的"建筑"（镜像）。

### 多阶段构建

> [!tip] 比喻：工厂的生产线
> 想象一个汽车工厂：
> - **第一阶段（Builder）**：在组装车间，使用各种重型设备、工具、原材料，制造出发动机、底盘等部件
> - **第二阶段（Runtime）**：在成品车间，只把制造好的部件组装成最终的汽车，不需要把所有工具和原材料都装进车里
>
> 多阶段构建类似：
> - 第一阶段使用完整的开发镜像，包含编译器、构建工具等
> - 第二阶段只复制编译产物到精简的运行时镜像
> - 最终镜像不包含构建工具，体积更小、安全性更高

### 最佳实践

```dockerfile
# 好的实践：先复制依赖文件，利用缓存
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .  # 代码变更不会导致依赖重新安装

# 坏的实践：一次复制所有
COPY . .
RUN pip install --no-cache-dir -r requirements.txt  # 任何代码变更都会重新安装依赖
```

> [!info] 缓存优化原则
> Docker 会逐层缓存构建结果。把**变化频率低**的操作放在前面，**变化频率高**的操作放在后面，可以最大化利用缓存，减少重复构建时间。

---

## 第七章 容器生命周期：从出生到轮回

### 状态流转

```
                    docker create
                         │
                         ▼
                    ┌─────────┐
                    │ created │
                    └────┬────┘
                         │
                    docker start
                         │
                         ▼
    docker unpause  ┌─────────┐   docker pause
         ┌──────────│ running │◄──────────┐
         │          └────┬────┘           │
         │               │                │
         │          ┌────▼────┐           │
         └─────────►│ paused  │───────────┘
                    └─────────┘
                         │
            docker stop / docker kill
                         │
                         ▼
                    ┌─────────┐
                    │ exited  │
                    └────┬────┘
                         │
                    docker rm
                         │
                         ▼
                    (已删除)
```

> [!tip] 比喻：人的生命周期
> 容器的生命周期类似人的生命周期：
> - **created（出生）**：婴儿出生，但还没有开始做事
> - **running（工作）**：成年人，正在工作、生活
> - **paused（休眠）**：暂时休息，可以随时醒来继续工作
> - **exited（退休）**：停止工作，但还在世（容器对象还在）
> - **rm（去世）**：彻底离开，所有痕迹被清除

> [!warning] 关键区别
> 停止的容器仍然占用磁盘空间（就像退休的人还占着房子），必须 `docker rm` 才能释放资源。

### 争议性问题：docker exec 是否是最佳调试方式？

**docker exec 的局限性：**
- 生产环境中不推荐开启 exec 功能（安全风险）
- 某些精简镜像没有 shell（如 scratch、distroless）
- 修改容器内文件不会持久化

**替代方案：**

| 方案 | 适用场景 |
|------|----------|
| `docker logs` | 查看应用日志（推荐首选） |
| `docker cp` | 复制文件出来分析 |
| 专用调试容器 | 共享 PID/Network Namespace |
| Kubernetes ephemeral containers | K8s 环境下的临时调试容器 |

---

## 第八章 数据持久化：保险箱、书桌与临时储物柜

### 三种存储方式

| 特性 | Volume（数据卷） | Bind Mount（绑定挂载） | tmpfs（内存挂载） |
|------|------------------|----------------------|-------------------|
| **存储位置** | Docker 管理的目录 | 宿主机任意路径 | 宿主机内存 |
| **持久性** | 是 | 是 | 否 |
| **Docker 管理** | 是 | 否 | 否 |
| **主要用途** | 生产环境数据 | 开发环境源码 | 临时敏感数据 |

> [!tip] 比喻：三种储物方式
> - **Volume（保险箱）**：银行的保险箱，由银行管理，安全可靠，即使你搬家（删除容器），保险箱里的东西还在
> - **Bind Mount（家里的书桌）**：你家里的书桌，你可以直接看到和修改上面的东西，但需要你自己管理
> - **tmpfs（酒店的临时储物柜）**：酒店房间里的保险柜，退房（容器停止）后东西就没了，适合放贵重但临时的物品

### 争议性问题：tmpfs 的实际应用场景

**适用场景：**
- 存储敏感信息（密码、令牌），不希望写入磁盘
- 高性能临时缓存（如 Redis 的 AOF 缓存）
- 容器内的临时文件系统（如 `/tmp`）

**不适用场景：**
- 需要持久化的数据
- 需要在容器间共享的数据
- macOS/Windows 环境（仅 Linux 支持）

---

## 第九章 Docker 网络：公寓楼的内部交换机

### 网络驱动类型

| 驱动 | 说明 | 使用场景 |
|------|------|----------|
| **bridge** | 默认驱动，创建虚拟网桥 | 单机多容器通信 |
| **host** | 共享宿主机网络栈 | 需要最高网络性能 |
| **none** | 完全禁用网络 | 安全隔离场景 |
| **overlay** | 跨主机通信 | Docker Swarm 集群 |
| **macvlan** | 容器拥有独立 MAC 地址 | 需要容器在局域网中可见 |

### bridge 网络

> [!tip] 比喻：公寓楼的内部交换机
> 想象公寓楼有一个内部电话交换机：
> - **默认 bridge（老式交换机）**：只能通过房间号（IP地址）打电话，没有通讯录
> - **自定义 bridge（智能交换机）**：有通讯录功能，可以直接拨打住户姓名（容器名），自动解析电话号码
>
> 自定义网络的关键优势：
> - 支持 DNS 解析（容器名自动解析为 IP）
> - 项目级隔离（不同项目的容器互不干扰）
> - 支持运行时连接/断开

### 最佳实践

```bash
# 始终使用自定义网络
docker network create mynet
docker run -d --name web --network mynet nginx
docker run -d --name db --network mynet postgres:16

# 可以直接通过容器名访问
docker exec web ping db  # 自动 DNS 解析！
```

### 端口映射安全

| 地址 | 含义 | 安全性 |
|------|------|--------|
| `0.0.0.0:8080` | 监听所有接口 | 较低（外部可访问） |
| `127.0.0.1:8080` | 只监听 localhost | 高（仅本机） |

> [!warning] 关键点
> 容器内服务必须监听 `0.0.0.0` 才能接受来自容器外部的连接。如果只监听 `127.0.0.1`，即使做了端口映射，外部也无法访问。

---

## 第十章 Docker Compose：乐队指挥的登台

### Docker Compose 是什么？

> [!tip] 比喻：乐队指挥
> 想象一个交响乐团：
> - **每个容器 like 一个乐手**：各自演奏自己的部分
> - **compose.yml like 乐谱**：定义每个乐手的演奏内容、何时开始、如何配合
> - **docker compose up like 指挥家的起拍手势**：一声令下，所有乐手同时开始演奏
> - **Docker Compose like 指挥家**：协调所有乐手，确保他们按照乐谱和谐地演奏
>
> 没有指挥家（Docker Compose），你需要逐个告诉每个乐手何时开始、如何配合，效率极低。

### 核心配置

```yaml
services:
  web:
    image: nginx
    ports:
      - "8080:80"
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

### ⚠️ 核心心法：容器互访用服务名，不是 localhost

这是 Compose 新手最容易踩的坑，值得单独强调。

```
# docker-compose.yml 中定义了两个服务：web 和 api
services:
  web:
    image: nginx
    ports:
      - "8080:80"
  api:
    image: my-api:latest
    ports:
      - "3000:3000"
```

在这个例子里：
- `web` 容器要访问 `api` 容器 → 用 `http://api:3000`
- `api` 容器要访问 `web` 容器 → 用 `http://web:80`
- **不能写 `http://localhost:3000`** → 那样访问的是容器自己，不是对方

> [!tip] 比喻：公寓楼的智能通讯录
> Docker Compose 为每个项目自动创建自定义 bridge 网络，这个网络内置了 **DNS 通讯录**：
> - 服务名 就是 房间号（比如 `api`、`web`、`db`）
> - DNS 自动把服务名翻译成容器的内部 IP
> - 你不需要知道 IP 是多少，直接喊名字就行
>
> 而 `localhost` 像你在自己房间里喊"喂"，只有你自己能听到——它永远指向**当前容器自己**。

> [!warning] 常见误区
> | 写法 | 实际访问 | 结果 |
> |------|----------|------|
> | `http://api:3000` | 访问 `api` 服务容器 | ✅ 正确 |
> | `http://localhost:3000` | 访问容器自己 | ❌ 访问了自己，对方收不到 |
> | `http://127.0.0.1:3000` | 访问容器自己 | ❌ 同上 |
> | `http://192.168.x.x` | 访问宿主机或外部 | ❌ 绕过了 Docker 网络 |

> [!info] 补充
> - 端口号要写**容器内部端口**，不是宿主机映射端口（`api` 服务内部监听 3000，就用 `:3000`，不是映射到宿主机的端口）
> - 服务名就是 `docker-compose.yml` 里 `services:` 下的 **key**（YAML 的键名）
> - 这个机制依赖 Compose 默认创建的自定义网络，如果你手动加入其他网络，需要确保 DNS 配置正确
> - 相关考点：[[课程回顾重点#考试回顾]]

### 健康检查与服务依赖

> [!tip] 比喻：接力赛的交接棒
> 想象接力赛：
> - **健康检查 like 运动员就位信号**：只有当前一棒的运动员准备好（服务健康），下一棒才能开始跑
> - **depends_on like 交接棒规则**：定义了"必须等前一棒准备好了才能开始"的规则
> - **condition: service_healthy like 具体的就位标准**：不仅是"站到起跑线"，而是"已经准备好接棒"

### 环境变量管理

**优先级：**
```
compose.yml environment > .env 文件 > 宿主机环境变量
```

---

## 第十一章 Podman 与其他工具：个人厨师 vs 餐厅

### Podman vs Docker

> [!tip] 比喻：个人厨师 vs 餐厅
> - **Docker like 餐厅**：有一个中央厨房（Daemon），所有菜品都从这里出。如果厨房出问题，整个餐厅停摆
> - **Podman like 个人厨师**：每个厨师独立工作（无守护进程），一个厨师出问题不影响其他人

### 功能对比

| 维度 | Docker | Podman |
|------|--------|--------|
| **守护进程** | 需要 dockerd daemon | 无守护进程 |
| **Root 权限** | daemon 默认需要 root | 支持 rootless |
| **安全风险** | daemon 是单点攻击目标 | 无中心 daemon，攻击面更小 |
| **Docker Compose** | 原生支持 | 支持（podman-compose） |
| **Pod 支持** | 不支持 | 原生支持 |

### 争议性问题：Overlay2 在生产环境的稳定性

**客观分析：**
- Overlay2 是 Docker 默认的存储驱动，经过多年验证
- 在大多数生产环境中表现稳定
- 极端情况下（大量小文件、频繁写入）可能出现性能问题
- 建议：对于写密集型应用，使用 Volume 绕过 CoW 开销

---

## 第十二章 安全最佳实践：修仙者的护体真气

### 容器安全清单

| 措施 | 说明 |
|------|------|
| 使用非 root 用户 | Dockerfile 中使用 `USER` 指令 |
| 最小化镜像 | 使用 slim/alpine 基础镜像 |
| 多阶段构建 | 不在最终镜像中包含构建工具 |
| 只读文件系统 | `--read-only` 标志 |
| 限制资源 | 使用 `--memory`、`--cpus` 限制 |
| 扫描漏洞 | 使用 Trivy、Snyk 等工具扫描镜像 |
| 不挂载 docker.sock | 避免容器获得宿主机 root 权限 |

### Docker Daemon 安全

**风险：**
- Daemon 默认以 root 权限运行
- 挂载 `docker.sock` 的容器可以获得宿主机 root 权限

**缓解措施：**
- 使用 Podman 的 rootless 模式
- 限制能够访问 docker.sock 的用户
- 使用 rootless Docker（Docker 20.10+）

---

## 附录：命令速查表

### 镜像操作

| 命令 | 说明 |
|------|------|
| `docker build -t name:tag .` | 构建镜像 |
| `docker images` | 列出本地镜像 |
| `docker rmi image_name` | 删除镜像 |
| `docker image prune` | 清理无用镜像 |
| `docker tag old:tag new:tag` | 给镜像打标签 |
| `docker push name:tag` | 推送镜像到仓库 |
| `docker pull name:tag` | 拉取镜像 |

### 容器操作

| 命令 | 说明 |
|------|------|
| `docker run -d --name myapp image` | 后台运行容器 |
| `docker run -it image /bin/bash` | 交互式运行 |
| `docker ps` | 查看运行中的容器 |
| `docker ps -a` | 查看所有容器 |
| `docker start/stop/restart container` | 启停容器 |
| `docker rm container` | 删除容器 |
| `docker logs -f container` | 实时查看日志 |
| `docker exec -it container bash` | 进入容器 |
| `docker cp container:/path /local/path` | 从容器复制文件 |
| `docker stats` | 查看资源使用 |
| `docker inspect container` | 查看详细信息 |

### 网络操作

| 命令 | 说明 |
|------|------|
| `docker network create mynet` | 创建网络 |
| `docker network ls` | 列出网络 |
| `docker network connect mynet container` | 连接容器到网络 |
| `docker network disconnect mynet container` | 断开网络 |

### 数据卷操作

| 命令 | 说明 |
|------|------|
| `docker volume create mydata` | 创建数据卷 |
| `docker volume ls` | 列出数据卷 |
| `docker volume rm mydata` | 删除数据卷 |
| `docker volume prune` | 清理无用卷 |

### Docker Compose 操作

| 命令 | 说明 |
|------|------|
| `docker compose up -d` | 后台启动所有服务 |
| `docker compose down` | 停止并删除所有服务 |
| `docker compose ps` | 查看服务状态 |
| `docker compose logs -f` | 实时查看日志 |
| `docker compose build` | 构建服务镜像 |
| `docker compose exec service bash` | 进入服务容器 |

---

## 相关概念

- [[wiki/Docker/容器与虚拟机]]
- [[wiki/Docker/Docker架构]]
- [[wiki/Docker/Docker镜像与分层]]
- [[wiki/Docker/Dockerfile]]
- [[wiki/Docker/容器生命周期与管理]]
- [[wiki/Docker/容器数据持久化]]
- [[wiki/Docker/Docker网络]]
- [[wiki/Docker/Docker Compose]]
- [[wiki/Docker/Podman与其他工具]]

## 来源

- Docker Docs: https://docs.docker.com/
- Podman Official: https://podman.io/
- gVisor: https://gvisor.dev/
- Kata Containers: https://katacontainers.io/
