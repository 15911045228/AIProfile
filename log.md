# 操作日志

## 2026-07-17

### 修整 | wiki 系统化改进（过期标记 + 交叉链接 + 部署清单）

- 更新 AIProfile/CLAUDE.md schema：
  - 页面规范增加 `status` 字段（active/deprecated/superseded）
  - 标签从单标签改为多标签
  - 摄入流程新增：标记 superseded 旧笔记 + 添加交叉链接
  - 增加 `wiki/checklists/` 目录
  - Lint 增加过期/重合/清单过时检查
- 全局 CLAUDE.md 同步更新操作指南
- 修整 7 个现有 wiki 页面：添加 status: active + 多标签 + 修复死链
- 创建 wiki/checklists/wechat-ai-deploy.md：企微客服完整部署清单
- 创建 raw/archive/ 目录

## 2026-07-16

### 修复 | 新增安全规则（密钥不入库）

- 发现 wiki 规则缺失安全约束，AIProfile 是 public 仓库，密钥暴露风险高
- 全局 CLAUDE.md 新增"安全前置"：严禁存储 API Key / Token / 密码 / 私钥 / 个人隐私
- AIProfile/CLAUDE.md 新增"安全规则（最高优先级）"章节
- 扫描现有 wiki 内容，未发现真实密钥值

## 2026-07-16

### 摄入 | 微信客服本地验证阶段0 + faiss 中文路径坑

- 阶段0 去风险全过：import app、mock /wxkf 端到端 4/4、BGE 下载 + FAISS 建库 42 块 + get_context 检索通
- 踩坑：faiss C++ 层在 Windows 中文路径下读写崩溃，修复 rag/store.py（临时文件中转）
- 保存原始笔记至 raw/notes/2026-07-16-wechat-kf-phase0-faiss.md
- 更新 wiki/projects/wechat-ai-service/overview.md — 本地验证状态更新为阶段0完成 + 新增 faiss 本地运行坑
- 更新 index.md（最后更新日期 2026-07-16）

### 摄入 | 真实 WECHAT_KF_* 凭证本地预检
- 用户提供了复用自建应用（2026-07-09 那个）的 CorpId/Secret/Token/AESKey
- 本地预检 ALL PASS：WeChatKF 用真实值构造成功（AESKey 合法 32 字节）、verify_url + parse_event 加解密往返通过
- 说明：仅证明凭证自洽、格式合法；live GET /wxkf 仍待云服务器 + 可信 IP + 企微后台触发
- 凭证仅存本地 .env（已 gitignore），未入 wiki

### 摄入 | 微信客服复用自建应用 + 服务器部署经验
- 重要实践：微信客服不需要新建自建应用，可直接复用已有的，改回调为 /wxkf 即可
- 服务器部署完成（腾讯云轻量 4C4G / Ubuntu 22.04 / IP 192.144.186.112）
- SSH 踩坑：MaxStartups 限流需间隔 8-10 秒重连
- Nginx 已配 /wxkf 路由、SSL 已生效（Let's Encrypt）、服务已重启、health OK
- 保存原始笔记至 raw/notes/2026-07-16-wechat-kf-deploy.md
- 更新 wiki/projects/wechat-ai-service/overview.md — 增加复用应用说明、更新服务器配置、更新待完成列表

### 摄入 | 数据库迁移 + 知识库升级到 pgvector
- 完成 PostgreSQL + pgvector 全栈迁移：6 张表（users、conversations、messages、kf_cursors、ai_logs、knowledge_chunks）
- 游标从 JSON 文件迁移到 PostgreSQL，解决多 workers 竞态
- 聊天记录自动落盘，重启不丢上下文
- FAISS 文件 → pgvector 向量搜索（`<=>` 余弦距离，精确搜索）
- 知识库 42 块从 `.vector_store/` 迁入 PostgreSQL
- 新增 cross-encoder reranker（BGE-reranker-v2-m3，HF 国内被墙，fallback 正常）
- 新增 ai_logs 表，每次 AI 调用记录耗时 + 模型 + token
- 保存原始笔记至 raw/notes/2026-07-17-pgvector-migration.md
- 更新 wiki/projects/wechat-ai-service/overview.md — 验证状态更新为 PG 迁移完成
- 更新 index.md 日期

## 2026-07-14

### 摄入 | Windows C 盘满盘清理

- 实战：本机 C 盘 80G 满盘(0 可用)排查 + 安全清理，释放 ~3.8G 恢复到 3.9G 可用
- 关键经验：Git Bash 下 `powershell -Command` 会吞掉 `$` 变量，须写 .ps1 用 `-File` 跑
- 关键认知：C 盘大头是 AppData 数据(37G)而非程序二进制(15G)，游戏多在别的盘
- 安全清理清单：WPS 旧安装包 / updater 残留 / NVIDIA 着色器缓存 / .gradle / npm-cache / 磁盘清理 / 关休眠
- 迁移思路：微信文件管理改路径最安全；mklink /J 迁 AppData 大目录收益最大
- 保存原始笔记至 raw/notes/2026-07-14-windows-c-disk-cleanup.md
- 创建 wiki/concepts/windows-c-disk-cleanup.md
- 更新 index.md（Concepts 新增入口）

### 摄入 | 微信客服接入加密自测 + 踩坑

- 本地 worktree 真实跑通微信客服加解密（verify_url + parse_event，5/5 通过）
- 踩坑：wechatpy 加密后端(cryptography)是可选依赖；入站回调信封与回复信封对称、可用 encrypt_message 造测试信封
- 保存原始笔记至 raw/notes/2026-07-14-wechat-kf-crypto-verify.md
- 更新 wiki/projects/wechat-ai-service/overview.md — 新增加密后端坑、入站信封测试技巧、本地验证状态小节
- 更新 index.md（最后更新日期 2026-07-14）

## 2026-07-10

### 摄入 | 拼豆店客服知识库

- 对话中完整规划了 15 个文件的客服知识库结构（三层：基础信息/问答服务/运营增强）
- 关键方法：意图库 + 案例库 + 复盘库的三层升级，价格与活动合并、历史活动沉淀
- 保存原始笔记至 raw/notes/2026-07-10-pindou-kb-structure.md
- 创建 wiki/projects/pindou-knowledge-base/overview.md
- 更新 index.md

### 摄入 | BGE Embedding + FAISS 向量 RAG 实战经验

- RAG 演进总结：TF-IDF(V1) → n-gram 哈希(V2) → BGE(V3)，最终方案为主流向量 RAG
- 踩坑记录：HuggingFace 被墙、HF-Mirror 不稳定、ModelScope 为最佳方案（~10MB/s）
- WeChatCrypto 参数名坑（corp_id → corpid → app_id），decrypt_echostr 方法缺失
- 保存原始笔记至 raw/notes/2026-07-10-bge-rag-practice.md
- 更新 wiki/projects/wechat-ai-service/overview.md — 增加 RAG 演进三阶段、新架构、15 文件知识库
- 更新 wiki/comparisons/rag-approaches.md — 增加 BGE 方案对比、HuggingFace 被墙解决方案、BGE 选型表
- 更新 index.md（无需新增入口，已有页面内容更新）

---

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