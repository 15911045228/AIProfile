---
tags: [project]
created: 2026-07-09
updated: 2026-07-16
sources: [claude-code-session]
---

# 企业微信 AI 客服搭建

## 概述

基于**企业微信自建应用** + **DeepSeek LLM** + **RAG 知识库**搭建的拼豆店 AI 客服系统。客户在企微发消息，AI 自动回复店铺信息（营业时间、价格、地址、活动等）。

项目目标：落地可用 + AI 作品集 + 练习 MCP/RAG + 可复制给其他店铺。

## 技术架构

```
用户 → 企业微信 → 企微服务器 → Nginx(SSL) → FastAPI → DeepSeek API + RAG → 回复
```

### 技术选型

| 层级 | 选型 | 原因 |
|------|------|------|
| 服务器 | 腾讯云轻量 4C4G / Ubuntu 22.04 | 99元/年，性价比高 |
| 反向代理 | Nginx + Let's Encrypt SSL | 免费证书，自动续期 |
| Web 框架 | FastAPI + Uvicorn | Python 异步，性能好 |
| 企微 SDK | wechatpy 1.8.18 | 处理加解密协议 |
| LLM | DeepSeek (deepseek-chat) | 便宜、国内直连 |
| 向量数据库 | FAISS (IndexFlatIP) | 高性能余弦相似度搜索 |
| Embedding | BAAI/bge-small-zh-v1.5 | 国产中文语义模型，维度 512 |
| 模型下载 | ModelScope（魔搭） | 国内镜像，下载速度 ~10MB/s |

## RAG 演进三阶段

| 阶段 | 方案 | 检索效果 | 部署速度 | 面试价值 |
|------|------|---------|---------|---------|
| V1 | TF-IDF + 余弦相似度 | 仅关键词匹配 | 秒级 | 低 |
| V2 | n-gram 哈希 + FAISS | 略好于 TF-IDF | 秒级 | 中（架构设计） |
| V3 | BGE Embedding + FAISS | 语义理解 | 需下载模型 | 高（主流方案） |

### V1→V2 的教训
hash embedding 是自己实现的，面试时无法解释"为什么不用主流方案"。一定要一步到位用成熟模型。

### V2→V3 的教训
- 不要用 HuggingFace 下载模型（国内被墙，下载速度 <20KB/s）
- 方案 A：ModelScope 国内镜像下载，实测 ~10MB/s
- 方案 B：直接下载 ONNX 模型（需解决网络问题）
- 最终选了方案 A：BGE 通过 ModelScope 走 sentence-transformers

## 部署要点

### 企微回调接入
- 注册企业微信 → 创建自建应用 → 配置回调 URL
- 回调验证 GET：验证 msg_signature + 解密 echostr
- 消息处理 POST：解密 XML → 处理 → 加密回复 XML
- 5 秒内必须响应

### WeChatCrypto 踩坑
- 参数名用 `app_id`（不是 `corp_id` 或 `corpid`）
- 没有 `decrypt_echostr` 方法，需自己调用 PrpCrypto 解密
- `decrypt_message` 参数顺序：`(msg, signature, timestamp, nonce)`
- **加密后端是可选依赖**：`pip install wechatpy` 不会自动装 `cryptography`/`pycryptodome`，运行时会报 `You must install either cryptography or pycryptodome!`，需 `pip install cryptography`
- **入站回调信封与回复信封对称**：本地自测微信客服 POST 回调时，可直接用 `WeChatCrypto.encrypt_message(inner_xml, nonce, timestamp)` 生成标准入站信封（自带正确 `MsgSignature`），再用 `parse_event` 解密抽取，避免手搓 CDATA 信封导致签名对不上

### 服务器配置
- 腾讯云防火墙需手动开放 80/443 端口（默认只开了 22 和 ICMP）
- Ubuntu 默认用户名 `ubuntu`（不是 `root`）
- 使用 systemd 管理进程，`EnvironmentFile` 加载 `.env`

## 知识库架构

15 个文件三层架构，从拼豆店知识库会话产出：

### 基础信息层（01-05）
店铺事实：介绍、价格、营业时间、预约、路线

### 问答服务层（06-11）
FAQ + 话术规范 + 营销引导 + 敏感问题处理

### 运营增强层（12-15）
意图识别、案例沉淀、复盘改进、更新记录

## 本地验证状态（2026-07-16）

在本地 worktree（E 盘 venv，依赖全装在 E 盘不碰 C 盘）完成阶段 0 去风险：

- **加解密真实往返 5/5 通过**（自生成 AESKey + wechatpy 真实 PrpCrypto/WeChatCrypto）
- **`import app` 全量导入通过**，路由齐全：`/wxkf`(GET+POST)、`/wechat/callback`(GET+POST)、`/health`
- **mock 版 `/wxkf` 端到端 4/4 通过**：POST 立即回 `success` → 后台 `sync_msg` 拉消息
  → `ai_service.chat` → `send_msg` 回发外部客户（桩掉企微网络与 LLM 调用）
- **BGE 模型下载 + FAISS 知识库构建通过**：`build_knowledge_base()` 入库 42 个知识块；
  `get_context("营业时间")` 检索返回 783 字符上下文，RAG 检索链路通
- 依赖栈已装全：torch 2.13 / sentence-transformers 5.6 / faiss-cpu 1.14 / modelscope 1.38 / fastapi / uvicorn / openai

### 本地运行踩坑（Windows + 中文路径开发机）
- **faiss 读写中文路径在 Windows 上会崩**：faiss 的 C++ 层用窄字符 `fopen`，
  UTF-8 中文路径被认错成"目录不存在"。表现为首次 `build_knowledge_base()` 写
  `.vector_store/index.faiss` 时 `RuntimeError: could not open ... for writing`。
  修复：`rag/store.py` 的 `_save`/`_load` 改为「先写/拷到 ASCII 临时文件再移入
  存储目录」（Linux 部署路径无中文，无副作用）。**Linux 服务器上不会触发此问题。**
- （此前已记）wechatpy 加密后端 `cryptography` 是可选依赖；入站回调信封与回复信封对称。

**尚未验证**（需真实凭证 / 公网 / 服务器）：
- 真实 `WECHAT_KF_*` 凭证下的 `GET /wxkf` 端到端（日志出 `[wxkf] URL 验证成功`）
- `ai_service` 接真实 LLM API key 的对话效果
- `kf/send_msg` 真实网络调用（需公网 IP + 可信 IP 配置）
- 自建应用通道（`wechat_handler.py`）自测
- 提交 git / 部署到腾讯云 `/opt/wechat-ai-service`

## 成本

| 项目 | 费用 |
|------|------|
| 服务器 | 99元/年 |
| 域名 (alexss.cn) | 64元/年 |
| SSL | 免费 |
| DeepSeek API | 按量，MVP 约几十元/月 |

## 项目目录结构

```
/opt/wechat-ai-service/
├── app.py              # FastAPI 入口
├── config.py           # 环境变量配置
├── wechat_handler.py   # 企微自建应用加解密
├── wechat_kf.py        # 微信客服通道（transport + 编排）
├── ai_service.py       # LLM 对话 + RAG 集成
├── rag/                # RAG 引擎包
│   ├── __init__.py     # 对外接口
│   ├── embedder.py     # 文本向量化（Strategy 模式）
│   ├── store.py        # FAISS 向量库
│   ├── engine.py       # 编排层
│   └── knowledge.py    # 知识库加载
├── knowledge/          # 15 个知识库 .md 文件
├── deploy/             # Nginx + systemd
├── .env                # 敏感配置
├── .model_cache/       # BGE 模型缓存
└── .vector_store/      # FAISS 索引持久化
```

## 关联页面

- [[rag-approaches]]
- [[pindou-content-ops]]
- [[pindou-knowledge-base]]
