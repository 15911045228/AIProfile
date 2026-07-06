---
tags: [project]
created: 2026-07-06
updated: 2026-07-06
sources:
  - raw/articles/2026-07-06-karpathy-llm-wiki.md
---

# 项目：LLM Wiki 搭建

## 项目背景

基于 Karpathy 的 LLM Wiki 模式，在 GitHub 上搭建个人 AI 工程能力知识库，实现知识的持续积累与复用。

## 技术选型

| 维度 | 选择 | 理由 |
|------|------|------|
| 仓库 | GitHub (SSH) | 版本管理 + 远程同步 |
| 格式 | Markdown | 纯文本、Obsidian 原生支持 |
| 本地工具 | Obsidian | Graph View / 双向链接 |
| Schema | CLAUDE.md 内嵌 | Claude Code 原生读取 |
| 组织方式 | concepts/projects/comparisons/syntheses | 匹配用户学习风格 |

## 目录结构

```
AIProfile/
├── CLAUDE.md           # Wiki 宪法
├── index.md            # 目录
├── log.md              # 操作日志
├── raw/                # 原始资料（只读）
└── wiki/               # Wiki 页面
    ├── concepts/       # 概念页
    ├── projects/       # 项目文档
    ├── comparisons/    # 方案对比
    └── syntheses/      # 综合综述
```

## 搭建步骤

1. 克隆空仓库 → 建目录
2. 写 CLAUDE.md (schema)
3. 创建 index.md + log.md
4. 安装 Obsidian + Dataview 插件
5. 第一次摄入（个人 Profile）
6. 当前项目 CLAUDE.md 中注册 wiki 路径

## 关联页面

- [[llm-wiki]]