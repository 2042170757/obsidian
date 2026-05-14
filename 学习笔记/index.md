# 学习笔记 - 知识库索引

基于 [Karpathy LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) 模式构建的个人知识库。

## 分类目录

### AI
- [[wiki/AI/]]
- AI 相关概念、论文、工具笔记
  - [[wiki/AI/karpathy-llm-wiki]] - Karpathy LLM Wiki 知识库详解
  - [[wiki/AI/大模型本质与工作原理]] - 概率预测、自注意力机制、自回归生成
  - [[wiki/AI/大模型核心参数]] - Token、上下文窗口、Temperature、Top-k/p、惩罚参数
  - [[wiki/AI/大模型训练流程]] - 预训练、SFT、RLHF、LoRA 训练全流程
  - [[wiki/AI/RAG技术入门与切片实战]] - RAG 原理、六种切片策略、Rerank 重排序
  - [[wiki/AI/模型量化与蒸馏]] - 量化精度、显存计算、知识蒸馏

### Network
- [[wiki/Network/]]
- 计算机网络、HTTP 协议相关笔记
  - [[wiki/Network/计算机网络基础]] - OSI 模型、MAC/IP、DNS、子网掩码、网关
  - [[wiki/Network/HTTP网络技术完整指南]] - HTTP 网络技术综合指南
  - [[wiki/Network/HTTP基础]] - HTTP 协议定义、工作流程、URL 结构
  - [[wiki/Network/HTTP方法]] - HTTP 方法语义、RESTful、PATCH 幂等性
  - [[wiki/Network/HTTP状态码]] - 状态码分类、特殊状态码辨析
  - [[wiki/Network/CORS跨域]] - 同源策略、预检请求、跨域配置
  - [[wiki/Network/实时通信技术]] - SSE、WebSocket、技术选型

### Docker
- [[wiki/Docker/]]
- Docker、容器技术相关笔记

### Python
- [[Python3基本数据类型与内建函数]]
- Python3 基本数据类型、数据类型转换、字符串/列表/字典内建函数

### 文学
- [[wiki/文学/]]
- 文学作品阅读笔记、文学知识

---

## 文件夹结构

```
学习笔记/
├── .raw/                    # 原始来源（不可修改）
│   ├── articles/           # 文章
│   ├── papers/             # 论文
│   └── clips/              # 剪藏
│
├── wiki/                   # LLM 生成的 wiki
│   ├── index.md            # 本文件
│   ├── log.md              # 变更日志
│   ├── AI/                 # AI 相关
│   │   ├── karpathy-llm-wiki.md
│   │   ├── 大模型本质与工作原理.md
│   │   ├── 大模型核心参数.md
│   │   ├── 大模型训练流程.md
│   │   ├── RAG技术入门与切片实战.md
│   │   └── 模型量化与蒸馏.md
│   ├── Network/            # 网络相关
│   │   ├── 计算机网络基础.md
│   │   ├── HTTP基础.md
│   │   ├── HTTP方法.md
│   │   ├── HTTP状态码.md
│   │   ├── CORS跨域.md
│   │   └── 实时通信技术.md
│   ├── Docker/             # Docker 相关
│   └── 文学/               # 文学相关
│
└── CLAUDE.md               # Schema（AI维护指南）
```

---

> 💡 提示：使用 Ctrl+O 或 Cmd+O 快速搜索笔记