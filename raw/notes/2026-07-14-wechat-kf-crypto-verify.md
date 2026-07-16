# 微信客服接入加密往返自测 + 踩坑（2026-07-14）

## 背景

在本地 worktree 验证 wechat-ai-service 的微信客服通道加解密能否跑通。
全部 .py 已通过 `py_compile` 语法检查；重依赖（faiss / sentence-transformers /
modelscope）未安装，用 `sys.modules` 桩掉 `ai_service` 与 `config`，直接 import
真实的 `WeChatKF` 类来跑真实方法。

## 踩坑 1：wechatpy 加密后端是可选依赖

- `pip install wechatpy` **不会**自动安装 `cryptography` 或 `pycryptodome`
- 运行时直接报：`You must install either cryptography or pycryptodome!`
- 解决：`pip install cryptography`（或 pycryptodome）

## 踩坑 2：入站回调信封与回复信封对称

- 微信客服 POST 回调的 XML 信封，与「我们 → 企微」的回复信封是同一套 schema
- 本地自测时，用
  `WeChatCrypto.encrypt_message(inner_xml, nonce, timestamp)`
  直接生成标准**入站**信封（自带正确 `MsgSignature`），再喂给 `parse_event` 解密抽取
- 比手搓 `<Encrypt><![CDATA[...]]></Encrypt>` 信封更可靠（手搓容易因 CDATA /
  签名计算细节对不上，导致 `InvalidSignatureException`）

## 测试结论（5/5 通过）

- `verify_url` 解密回显 echostr（GET 验证往返）→ 还原出原文
- `verify_url` 错误签名 → 抛 `ValueError` 被拒
- `parse_event` 抽 `token` / `open_kfid`（POST 回调）→ 字段正确
- `parse_event` 错误签名 → 返回 `None`
- `parse_event` 非 `kf_msg_or_event` 事件 → 返回 `None`

## 环境约定（避免写 C 盘）

- venv 与 site-packages 放在 E 盘项目目录（不碰系统 C 盘）
- pip 下载缓存重定向到 E 盘（`PIP_CACHE_DIR`），安装后 `pip cache purge` 清掉
  C 盘默认 pip 缓存目录
