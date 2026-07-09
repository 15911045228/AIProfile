# 操作日志

## 2026-07-09

### 摄入 | 企业微信 AI 客服搭建

- 对话中完整实现了企业微信 AI 客服从零搭建：服务器选购 → 域名 SSL → 企微回调 → DeepSeek 接入 → RAG 知识库
- 保存原始笔记至 raw/notes/2026-07-09-wechat-ai-service-setup.md
- 创建 wiki/projects/wechat-ai-service/overview.md — 技术架构、部署要点、踩坑记录（WeChatCrypto 参数名问题）
- 更新 index.md — 新增项目入口

### 摄入 | RAG 方案对比

- 对比 TF-IDF 和向量数据库在小知识库场景下的优劣
- 保存原始笔记至 raw/notes/2026-07-09-rag-approaches-comparison.md
- 创建 wiki/comparisons/rag-approaches.md
- 更新 index.md — 首次填充 Comparisions 板块

### 修复 | 摄入前置改为不可跳过流程步骤

- 根因：LLM 在"任务模式"下不会主动跳出当前任务执行摄入检查
- 修复：摄入前置从"自动判断"改为"每完成一个独立任务，在汇报完成前必须执行摄入检查"
- lessons.md 补充第三次修复记录

---

## 2026-07-07

### 摄入 | Wiki 自动化缺陷分析

- 对话中实测发现 CLAUDE.md 的"主动询问"约束力不足，LLM 不会自动读取 wiki
- 保存原始分析至 raw/notes/2026-07-07-wiki-auto-access-flaw.md
- 更新 wiki/projects/llm-wiki-setup/lessons.md — 新增 LLM-Wiki 交互缺陷章节及修复方向建议
- 无需更新 index.md（已有指向 lessons.md 的链接）

### 修复 | 全局 CLAUDE.md 强化

- 针对发现的缺陷，CLAUDE.md 措辞从"建议性"改为"强制规则（前置约束）"
- 查询前置：回答前必须先读 index.md + 相关页面
- 摄入前置：产出有价值内容后自动判断，直接问用户
- lessons.md 补充修复实施记录

### 修复 | 跨轮复查 + 摄入节点明确化

- 采纳企微客服会话建议，增加跨轮复查机制：话题切换时重新检查 index.md
- 摄入前置增加五个具体触发节点（决策确认/功能完成/踩坑发现/方案对比/技术调研）
- lessons.md 补充第二次修复记录

### 初始化 | Wiki 骨架搭建

- 创建目录结构（raw/ + wiki/ 及其子目录）
- 创建 CLAUDE.md schema
- 创建 index.md 和 log.md
- 初始化 Git 仓库并推送至 GitHub

### 摄入 | 个人 Profile

- 保存原始资料至 raw/notes/2026-07-06-personal-profile.md
- 创建 wiki/syntheses/profile-summary.md
- 更新 index.md

### 摄入 | LLM Wiki 概念 + 搭建项目记录

- 保存 Karpathy 原文摘录至 raw/articles/2026-07-06-karpathy-llm-wiki.md
- 创建 wiki/concepts/llm-wiki.md 概念页
- 创建 wiki/projects/llm-wiki-setup/overview.md + lessons.md
- 更新 index.md

### 初始化 | 拼豆店内容运营追踪

- 创建 wiki/projects/pindou-content-ops/overview.md
- 创建 wiki/projects/pindou-content-ops/topic-tracker.md
- 创建 wiki/projects/pindou-content-ops/analysis.md
- 更新 index.md