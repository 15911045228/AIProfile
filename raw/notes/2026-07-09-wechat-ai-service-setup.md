---
tags: [project]
created: 2026-07-09
updated: 2026-07-09
sources: [claude-code-session]
---

# 企业微信 AI 客服搭建

## 概述

基于企业微信 + DeepSeek + RAG 搭建的拼豆店 AI 客服系统。客户在企业微信发消息 → AI 自动回复店铺信息。

## 技术架构

```
用户 → 企业微信 → 企微服务器 → Nginx(SSL) → FastAPI → DeepSeek API + RAG → 回复
```

| 层级 | 技术选型 |
|------|---------|
| 服务器 | 腾讯云轻量 4C4G3M / Ubuntu 22.04 |
| 反向代理 | Nginx + Let's Encrypt SSL |
| Web 框架 | FastAPI + Uvicorn |
| 企微 SDK | wechatpy |
| LLM | DeepSeek (deepseek-chat) |
| RAG | TF-IDF + 余弦相似度（轻量级，无模型依赖） |
| 进程管理 | systemd 守护 |

## 关键步骤

### 1. 环境准备
- 腾讯云购买轻量服务器（选 Ubuntu 22.04，不选收费镜像）
- 购买域名（alexss.cn），DNS 解析 A 记录指向服务器 IP
- 服务器防火墙开放 80/443 端口
- 安装 Python3、Nginx、certbot（Let's Encrypt SSL）

### 2. 企业微信接入
- 注册企业微信，创建自建应用
- 配置回调 URL：`https://域名/wechat/callback`
- Token 自定义，EncodingAESKey 随机生成
- 回调验证流程：GET 请求验证签名 + 解密 echostr

### 3. 企业微信加解密踩坑
- `WeChatCrypto` 参数名：`app_id`（不是 `corp_id` 也不是 `corpid`）
- 没有 `decrypt_echostr` 方法 → 需手动验证签名 + PrpCrypto 解密
- `decrypt_message` 参数顺序：`(msg, signature, timestamp, nonce)`

### 4. AI 接入
- OpenAI 兼容接口，配置 `base_url=https://api.deepseek.com/v1`
- 系统提示词定义客服角色
- RAG 自动检索知识库相关内容注入上下文

### 5. 生产部署
- systemd 服务 + 环境变量文件
- Nginx 反向代理 + SSL
- 日志通过 journalctl 查看

## 成本

| 项目 | 费用 |
|------|------|
| 服务器 | 99元/年 |
| 域名 | 64元/年 |
| SSL | 免费（Let's Encrypt） |
| DeepSeek API | 按量，MVP 约几十元/月 |

## 关联页面

- [[rag-approaches]]
- [[pindou-content-ops]]