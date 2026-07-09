---
tags: [comparison]
created: 2026-07-09
updated: 2026-07-09
sources: [claude-code-session]
---

# RAG 实现方案：TF-IDF vs 向量数据库

## 概述

RAG（Retrieval-Augmented Generation）是实现 AI 客服知识库的关键环节。对比两种方案在小规模知识库场景下的适用性。

## 方案对比

| 维度 | TF-IDF + 余弦相似度 | 向量数据库（ChromaDB + ONNX Embedding） |
|------|-------------------|--------------------------------------|
| 检索原理 | 关键词匹配 + TF 加权 | 语义 Embedding + 向量距离 |
| 额外依赖 | 无 | 需下载 ONNX 模型（~80MB） |
| 部署速度 | 秒级 | 首次下载需数十分钟 |
| 语义理解 | ❌ 只能匹配词汇 | ✅ 理解近义词/语义 |
| 小知识库（<100条） | 够用 | 够用 |
| 大知识库（>1000条） | 效果下降 | 效果好 |
| 短查询（2-4字） | 容易无匹配 | 更好 |

## TF-IDF 的不足

TF-IDF 本质是词汇重叠匹配，无法理解语义关联：
- 用户说"钜惠" → 匹配不到"优惠"
- 用户说"怎么玩" → 匹配不到"制作流程"
- "几点开门" vs "营业时间" → 字面不重叠则无匹配（可在知识库中加别名缓解）

## 什么时候该升级

- 知识库条目超过 100 条
- 用户问法多样，频繁出现"我说了但 AI 没理解"的情况
- 需要跨文档的语义关联检索

## 升级路径

升级只需重写 `knowledge_base.py`：

```python
import chromadb
client = chromadb.PersistentClient(path=".chroma_db")
collection = client.get_or_create_collection("shop_knowledge")
collection.add(documents=[...], ids=[...])
results = collection.query(query_texts=[user_msg], n_results=3)
```

Embedding 模型可选：
1. **本地 ONNX**（all-MiniLM-L6-v2）— 免费，首次需下载
2. **API Embedding**（DeepSeek / OpenAI）— 按量付费，无需下载

## 关联页面

- [[wechat-ai-service]]