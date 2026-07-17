# 微信客服接入实战 + 服务器部署记录（2026-07-16）

## 企微微信客服接入：不需要新建自建应用

之前官方文档/README 说"建一个专用自建应用"是为了避免两个通道抢回调地址。
但如果只做微信客服（不做内部员工通道），**直接复用已有的自建应用即可**：

1. 绑定微信客服账号到该应用（微信客服 → 创建账号 → 绑定）
2. 把该应用的回调 URL 从 `/wechat/callback` 改为 `/wxkf`
3. `.env` 的 `WECHAT_KF_*` 填同一个应用的 CorpId/Secret/Token/AESKey
4. 微信客服代码（wechat_kf.py）不读 AgentId，其他字段与自建应用通道完全一致

## 服务器部署流程（腾讯云轻量 + Ubuntu 22.04）

按顺序一次成功：

1. **备份旧版**：`sudo cp -a /opt/wechat-ai-service /opt/wechat-ai-service.bak`
2. **上传代码**：scp 逐一上传新文件（wechat_kf.py、cursor_store.py、app.py、config.py、ai_service.py、rag/、knowledge/、deploy/wechat-nginx.conf）
3. **合并 .env**：保留旧版 WECHAT_* + LLM_API_KEY，新增 WECHAT_KF_* 行
4. **装依赖**：`pip install -r requirements.txt`（服务器 venv 已预装 torch/transformers/faiss/sentence-transformers，补装 wechatpy/openai 等）
5. **更新 Nginx**：增加 `location /wxkf { proxy_pass ... }`，`sudo systemctl reload nginx`
6. **重启服务**：`sudo systemctl restart wechat-ai`
7. **验证**：`curl https://alexss.cn/health` → `{"status":"ok","app":"拼豆小助手","ai_configured":true}`

## SSH 踩坑：MaxStartups 限流

- 现象：连续快速 SSH 时，连接被 `Exceeded MaxStartups` 拒绝
- 根因：sshd_config 的 MaxStartups 防止未认证连接洪水，默认值较低
- 表现：`kex_exchange_identification: banner line 0: Exceeded MaxStartups`
- 解决：两次 SSH 之间等 8-10 秒，避免同时建立多个连接
- 注意：scp 也算一次 SSH 连接，同样受此限制

## 服务器配置速查

- IP: 192.144.186.112
- 域名: alexss.cn → 已配 Let's Encrypt SSL（2026-07-09 ~ 2026-10-07）
- SSH: ubuntu 用户，admin.pem 密钥认证
- 服务: systemd wechat-ai（uvicorn 2 workers, 127.0.0.1:8000）
- Nginx: 443 SSL → proxy 8000，路由 /wxkf、/wechat/callback、/external/callback、/health
- 可信 IP 待填: 192.144.186.112