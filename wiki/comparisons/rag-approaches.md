---
tags: [comparison, rag]
status: active
created: 2026-07-09
updated: 2026-07-10
sources: [claude-code-session]
---

# RAG 实现方案对比

## 概述

RAG（Retrieval-Augmented Generation）是实现 AI 客服知识库的关键环节。对比三种方案在实战中的表现。

## 方案对比

| 维度 | TF-IDF | n-gram 哈希 + FAISS | BGE Embedding + FAISS |
|------|--------|-------------------|----------------------|
| 检索原理 | 关键词匹配 | 字符级哈希碰撞 | 语义 Embedding |
| 额外依赖 | 无 | numpy + faiss | sentence-transformers + 模型文件 |
| 部署速度 | 秒级 | 秒级 | 需下载模型（~100MB，魔搭 ~10s） |
| 语义理解 | ❌ | ❌ | ✅ |
| 中文效果 | 差 | 差 | 好（BGE 专为中文优化） |
| 小知识库 | 能用 | 能用 | 好 |
| 面试价值 | 低 | 中（体现架构设计） | 高（主流方案） |

## 实战经验

### 不要用自制的 Embedding

自己写 hash embedding 看似"省了模型下载"，但面试时无法解释"为什么不用主流方案"。一步到位用 BGE 才是正确选择。

### HuggingFace 被墙怎么办

国内服务器无法直接访问 HuggingFace，速度 <20KB/s。解决方案：

1. **ModelScope（推荐）**：阿里旗下的模型托管平台，国内下载 ~10MB/s
2. **预下载后上传**：在本机下载好 ONNX 模型，scp 到服务器
3. **API Embedding**：使用阿里 DashScope / SiliconFlow 等 API

### 模型下载方式对比

| 方式 | 速度 | 稳定性 | 说明 |
|------|------|--------|------|
| HuggingFace 直连 | <20KB/s | ❌ 被墙 | 不可用 |
| HF-Mirror | ~20KB/s | ❌ 不稳定 | 很多时候也慢 |
| ModelScope | ~10MB/s | ✅ 稳定 | 推荐 |
| 本地下载 + scp | 取决于本机网络 | ✅ 最可靠 | 适合一次性部署 |

### BGE 模型选型

| 模型 | 维度 | 大小 | 中文效果 | 推荐场景 |
|------|------|------|---------|---------|
| bge-small-zh-v1.5 | 512 | ~100MB | 好 | MVP 首选 |
| bge-base-zh-v1.5 | 768 | ~400MB | 更好 | 生产环境 |
| bge-large-zh-v1.5 | 1024 | ~1.3GB | 最好 | 高精度场景 |

## 关联页面

- [[wechat-ai-service]]