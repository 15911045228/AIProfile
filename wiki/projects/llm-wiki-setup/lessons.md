---
tags: [project]
created: 2026-07-06
updated: 2026-07-06
---

# LLM Wiki 搭建踩坑记录

## Obsidian 自动生成文件

打开 vault 后 Obsidian 自动创建了 `.obsidian/` 目录和若干配置文件（core-plugins.json, graph.json, workspace.json）。其中 `workspace.json` 是会话状态，**应排除**，其他配置文件可以保留。另有一个空的 `mcp.md` 文件自动生成在根目录，原因不明，已删除。

## 修复：.gitignore 配置

初始 `.gitignore` 只排除了 `.obsidian/workspace`，但不匹配 `workspace.json`。更正为显式排除 `.obsidian/workspace.json`。

## SSH 认证

仓库使用 SSH 方式推送，需先确认 SSH key 已配置到 GitHub。

## 双仓库策略

项目（ClaudeWork）和 wiki（AIProfile）在不同仓库。需要在当前项目的 CLAUDE.md 中手动注册 wiki 路径，让 Claude 知道 wiki 的位置。

## 学到的经验

- Obsidian 打开空 vault 会自动生成配置，不是所有文件都要 commit
- 架构设计阶段先画目录树再写文件效率高
- CLAUDE.md 里的<!-- superpowers-zh -->标记之间是自动生成区，自定义内容要写在标记外