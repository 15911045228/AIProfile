---
tags: [article]
created: 2026-07-06
updated: 2026-07-06
---

# LLM Wiki — A pattern for building personal knowledge bases using LLMs

> 作者：Andrej Karpathy
> 原文：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f

## 核心思想

大多数 LLM 知识系统（RAG）每次查询都重新检索原始文档，没有积累。LLM Wiki 相反——LLM **增量构建并持续维护一个结构化的 wiki**，知识编译一次后保持更新，而非每次重新推导。

## 三层架构

1. **Raw sources（原始资料）** — 不可变，LLM 只读不写
2. **Wiki** — LLM 全权拥有的 MD 文件，负责创建/更新/交叉引用
3. **Schema** — 如 CLAUDE.md，规定 wiki 结构和工作流

## 三类操作

- **Ingest（摄入）** — 丢资料 → LLM 读 + 讨论 → 写页 → 更新 index → 追 log
- **Query（查询）** — 提问 → 读 index 定位 → 综合回答（好答案回填 wiki）
- **Lint（体检）** — 定期找矛盾、过时、孤儿页、缺口

## 关键洞察

知识库累人的是记账（更新交叉引用、保持一致性），不是读和想。LLM 不会烦不会忘，维护成本趋近于零。

## 可选工具

Obsidian（浏览）/ qmd（搜索）/ Marp（幻灯片）/ Dataview（动态查询）