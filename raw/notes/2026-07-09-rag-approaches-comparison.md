---
tags: [comparison]
created: 2026-07-09
updated: 2026-07-09
sources: [claude-code-session]
---

# RAG 实现方案对比

## 概述

RAG（检索增强生成）的实现方式对比：轻量 TF-IDF 方案 vs 标准向量数据库方案。

## TF-IDF 方案（当前使用）

| 维度 | 说明 |
|------|------|
| 检索原理 | 关键词匹配 + TF-IDF 权重 + 余弦相似度 |
| 依赖 | 纯 Python 标准库 |
| 部署速度 | 秒级 |
| 小知识库（< 100 条） | 效果够用 |
| 大知识库（> 1000 条） | 效果下降，无法理解语义 |
| 适用场景 | MVP 快速验证、知识库极小 |

## 向量数据库方案（未来升级）

| 维度 | 说明 |
|------|------|
| 检索原理 | 语义 Embedding + 向量相似度 |
| 依赖 | ChromaDB + 本地 ONNX 模型（all-MiniLM-L6-v2）或 API Embedding |
| 部署速度 | 慢（需下载 80MB ONNX 模型） |
| 小知识库 | 效果取决于 Embedding 质量 |
| 大知识库 | 语义搜索，效果好 |
| 适用场景 | 知识库规模大、需要语义理解 |

## TF-IDF 的局限性

TF-IDF 本质是**词汇重叠匹配**，问题在：
1. 用户说"钜惠" → 匹配不到"优惠"
2. 用户说"怎么玩" → 匹配不到"拼豆制作流程"
3. 短查询（2-4 字）匹配精度低

向量 Embedding 可以理解这种语义关联。

## 切换策略

从 TF-IDF 升级到向量版只需要重写 `knowledge_base.py` 这一个文件：

```python
# 伪代码
import chromadb
client = chromadb.PersistentClient(path=".chroma_db")
collection = client.get_or_create_collection("shop_knowledge")
# Embedding 在 add 和 query 时自动处理
collection.add(documents=[...], ids=[...])
results = collection.query(query_texts=[user_msg], n_results=3)
```

## 关联页面

- [[wechat-ai-service]]