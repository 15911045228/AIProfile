---
tags: [concept, wiki, llm]
status: active
created: 2026-07-06
updated: 2026-07-06
sources:
  - raw/articles/2026-07-06-karpathy-llm-wiki.md
---

# LLM Wiki

## 概述

LLM Wiki 是由 Andrej Karpathy 提出的个人知识库模式：由 LLM 持续维护的 Markdown wiki，知识编译一次后保持更新，而非每次查询重新检索原始文档。

## 核心要点

- 与传统 RAG 的关键区别：**知识编译一次，持续更新，而非每次重新推导**
- 三层架构：Raw sources（不可变）→ Wiki（LLM 全权拥有）→ Schema（配置规则）
- 三类操作：Ingest（摄入）/ Query（查询）/ Lint（体检）
- 关键洞察：LLM 处理记账（交叉引用、一致性维护），人类处理思考（策展、提问、分析）

## 为什么有效

人类维护 wiki 的痛点不是读和想，而是维护成本（更新交叉引用、保持一致性、协调多页面）随 wiki 增长而超线性增长。LLM 不会烦、不会忘、能一次改 15 个文件，**维护成本趋近于零**。

## 适用场景

- 个人知识管理（读书/研究/学习）
- 技术文档积累
- 项目经验沉淀
- 团队内部知识库

## 关联页面

- [[llm-wiki-setup]]

## 外部链接

- [原始 Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Obsidian](https://obsidian.md/)
- [qmd 搜索](https://github.com/tobi/qmd)