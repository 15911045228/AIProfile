---
tags: [project]
created: 2026-07-09
updated: 2026-07-09
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
| 企微 SDK | wechatpy | 处理加解密协议 |
| LLM | DeepSeek (deepseek-chat) | 便宜、国内直连 |
| RAG | TF-IDF + 余弦相似度 | 轻量无依赖，知识库小时够用 |

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

### 服务器配置
- 腾讯云防火墙需手动开放 80/443 端口（默认只开了 22 和 ICMP）
- Ubuntu 默认用户名 `ubuntu`（不是 `root`）
- 使用 systemd 管理进程，`EnvironmentFile` 加载 `.env`

## 知识库管理

知识库位于 `knowledge/*.md`，编辑后重启服务生效：

```bash
sudo systemctl restart wechat-ai
```

知识库分类：店铺信息、价格信息、常见问题。

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
├── wechat_handler.py   # 企微加解密
├── ai_service.py       # LLM 对话
├── knowledge_base.py   # RAG 检索
├── knowledge/          # 知识库内容（.md）
│   ├── shop_info.md
│   ├── pricing.md
│   └── faq.md
├── deploy/             # Nginx + systemd 配置
├── .env                # 敏感配置
└── requirements.txt
```

## 关联页面

- [[rag-approaches]]
- [[pindou-content-ops]]