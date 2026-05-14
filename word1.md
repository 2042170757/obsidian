容器解决了什么问题？

从能跑起来到到处乱跑，传统痛点的区别

容器 vs 虚拟机 ：有什么不一样？

虚拟机架构，容器架构，隔离机制，镜像体积，启动效率，资源开销

Docker 架构： Client-Server 模式：

Docker Daemon（dockered），Docker Client（docker），Dokcer Registry（镜像仓库），核心管理对象

Docker镜像操作：

只读的分层模板：

核心架构与特性：文件系统分层，高效共享与复用（CoW：copy on write），只读模板与可写容器

镜像命名与版本规范：标准命名格式：[仓库地址/] <用户名> / <镜像名> : <标签>，生产环境常用示例

镜像操作：搜索，拉取，查看，删除（各个命令）

搜索，docker search，拉取，docker pull，查看，docker images，删除docker rmi

Dockerfile ：定义你的镜像“菜谱”

python web 应用构建示例（举例）：

 1.指定轻量基础镜像 2.设置工作目录（推荐为绝对路径）3.优先复制依赖（利用缓存）4.复制应用代码&声明端口

Dockerfile 进阶：多阶段构建，镜像减肥神器

普通构建（举例），多阶段构建（举例）

构建镜像：docker build 与 docker tag

核心构建命令（举例）:-t,-f,--no-cache

标签管理与推理（..）:

构建上下文优化:

容器生命周期：从创建到销毁

容器状态流转逻辑：creat（已创建），

running←unpause→paused，running←start→exited,

强制终止：any→docker rm→exited，彻底销毁：exited→docker run →释放资源

核心状态定义解析：created，running，paused，exited

核心误区警示：停止不等于删除

docker run 核心参数：容器启动全功略：

-d，-it，--rm，-d，-p，-P，-e

基础运行模式，

端口映射，

命名与资源限制，

环境变量注入

常见端口：8080（HTTP，Web），3306（MySQL），22（SSH）

查看日常管理：查看，停止，删除，清理

查看容器状态：ps，ps -a，inspect，stats（举例）

停止与删除容器：stop，kill，rm（操作停止的容器），container，

系统资源一键清理：system prune（/-a）

调试利器：docker exec 与 docker logs

进入容器：1.进入交互式Bash（常用）-it，2.以root身份进入排查权限问题 docker exec -it --user root my-app sh

查看日志：docker logs -f my-nginx

容器数据持久化：

核心痛点：容器删除后其内部的数据会完全丢失，如何确保mysql等服务的数据安全保留？

tmpfs（内存挂载）

bind mount（绑定挂载）

docker volume（数据卷）

核心命令速查（bash）：

docker volume create my-db-data

docker run -d -v my-db-data:/var/lib/mysql mysql:8.0

docker run -d $(pwd)/src:/app my-app:dev

Docker网络：容器如何互相通信？

bridge（默认桥接）（重点），host（宿主机网络），none（无网络），

核心操作代码（自定义bridge网络）：1.创建自定义网络2.启动容器并加入网络3.容器间互访（无需IP）

核心优势：自动DNS解析

Docker Compose 多容器编排

为什么需要Docker Compose？

传统方法：手工管理多容器（易错且低效）（举例）

Compose 方法：YAML定义，一键启停

compose.yml：定义多容器应用的一切

核心三要素：结构清晰

安全依赖：健康检查

服务互联：自动DNS

Compose CLI 常用命令速查：

1.启动与停止服务：

启动所有服务（后台运行/构建镜像）

启动指定服务（包含依赖），例如web

停止服务/彻底清理（含数据卷）

2.服务状态与日志：

3.进阶运维操作：

Compose 环境变量管理：告别硬编码的配置

1.env环境配置文件（核心数据源）

2.compose.yml动态变量引用

3.灵活的多环境配置策略

4.安全与写作最佳实践

Podman：下一代无守护进程容器引擎

Docker架构（C/S守护进程模式）

Podman架构（无Daemon模式）

-d 后台运行

docker run = docker pull + docker start

lib -> usr/lib：（→）表示连接文件（usr表示系统资源）

lrwxrwxrwx：l表示软链接

docker export docker import

docker load docker save

dockers rm docker rmi

docker pull下来第一层是系统层，每加一层增加一层读写层

0.0.0.0:8080（0.0.0.0的含义）

0.0.0.0:8080->80/tcp, [::]:8080->80/tcp（含义）

基础命令：

docker pull

docker run -it -p -d

docker ps

docker start

docker restart

docker stop

docker exec -it

docker rm

docker logs

docker top