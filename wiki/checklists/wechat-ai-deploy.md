---
tags: [checklist, wechat, deploy]
status: active
created: 2026-07-17
updated: 2026-07-17
sources:
  - raw/notes/2026-07-09-wechat-ai-service-setup.md
  - raw/notes/2026-07-16-wechat-kf-deploy.md
  - raw/notes/2026-07-17-pgvector-migration.md
---

# 企微 AI 客服部署清单

> 新服务器从零到可用的全流程。每完成一项打勾。

## 1. 服务器准备

- [ ] **购买服务器**：腾讯云轻量 4C4G / Ubuntu 22.04（99元/年）
- [ ] **SSH 登录**：`ssh -i admin.pem ubuntu@<IP>`
- [ ] **域名解析**：A 记录指向服务器 IP
- [ ] **开放端口**：22(SSH) / 80(HTTP) / 443(HTTPS)

## 2. 基础环境

- [ ] **系统更新**：`sudo apt update && sudo apt upgrade -y`
- [ ] **Python 虚拟环境**：`python3 -m venv venv && source venv/bin/activate`
- [ ] **安装系统包**：
  ```bash
  sudo apt install -y nginx certbot python3-certbot-nginx postgresql postgresql-contrib
  ```
- [ ] **安装 Python 依赖**：
  ```bash
  pip install fastapi uvicorn[standard] wechatpy openai sentence-transformers psycopg2-binary
  ```
- [ ] **PostgreSQL 建库**：
  ```sql
  CREATE DATABASE wechat_ai;
  CREATE EXTENSION vector;
  ```

## 3. 应用部署

- [ ] **上传代码**：`scp -r ./* ubuntu@<IP>:/opt/wechat-ai-service/`
- [ ] **配置 .env**：填入 WECHAT_* + LLM_API_KEY + 数据库连接串
- [ ] **建表**：运行 `python -c "from app import init_db; init_db()"`
- [ ] **下载 BGE 模型**（ModelScope 国内镜像）：
  ```bash
  pip install modelscope
  python -c "from modelscope import snapshot_download; snapshot_download('BAAI/bge-small-zh-v1.5')"
  ```
- [ ] **构建知识库向量**：`python -c "from rag import rebuild_index; rebuild_index()"`
- [ ] **验证导入**：`curl http://127.0.0.1:8000/health` → `{"status":"ok"}`

## 4. Nginx + SSL

- [ ] **配置 Nginx**：复制 `deploy/wechat-nginx.conf` 到 `/etc/nginx/sites-enabled/`
  - `location /wxkf { proxy_pass http://127.0.0.1:8000; }`
  - `location /health { proxy_pass http://127.0.0.1:8000; }`
- [ ] **申请 SSL**：`sudo certbot --nginx -d alexss.cn`
- [ ] **测试 Nginx**：`sudo nginx -t && sudo systemctl reload nginx`

## 5. Systemd 服务

- [ ] **配置服务**：复制 `deploy/wechat-ai.service` 到 `/etc/systemd/system/`
- [ ] **启动服务**：`sudo systemctl enable wechat-ai && sudo systemctl start wechat-ai`
- [ ] **验证**：浏览器打开 `https://alexss.cn/health` → `{"status":"ok","app":"拼豆小助手","ai_configured":true}`

## 6. 企微回调配置

- [ ] **企业微信后台设置回调 URL**：`https://alexss.cn/wxkf`
- [ ] **设置可信 IP**：在企微后台的"可信 IP"列表中添加服务器 IP
- [ ] **微信客服绑定**：微信客服后台创建客服账号 → 绑定到该自建应用
- [ ] **验证回调**：企微后台"测试回调" → 返回 `success`

## 7. 上线验证

- [ ] **加解密真实往返**：真实微信用户发消息 → AI 回复
- [ ] **检查日志**：`sudo journalctl -u wechat-ai -f` 无报错
- [ ] **检查数据库**：确认 `messages` 表有数据写入
- [ ] **监控 DeepSeek API 调用**：确认 `ai_logs` 表有 token 用量记录

## 8. 运维备忘

- [ ] SSL 自动续期（certbot 默认已配 systemd timer）
- [ ] SSH 连接之间等 8-10 秒（MaxStartups 限流）
- [ ] 不可同时跑多个 scp 连接

## 关联页面

- [[wechat-ai-service]]
- [[pindou-knowledge-base]]
- [[rag-approaches]]