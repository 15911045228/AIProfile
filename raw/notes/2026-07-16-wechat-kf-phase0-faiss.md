# 微信客服本地验证阶段0 + faiss 中文路径坑（2026-07-16）

## 阶段0 去风险结果（本地 worktree，E 盘 venv）
- 全量依赖装 E 盘 venv（torch/sentence-transformers/faiss-cpu/modelscope/fastapi/uvicorn/openai），不碰 C 盘
- import app 通过；路由 /wxkf、/wechat/callback、/health 齐全
- mock /wxkf 端到端 4/4：POST 回 success → 后台 sync_msg 拉消息 → chat → send_msg 回发（桩掉网络+LLM）
- BGE 模型 ModelScope 下载 + build_knowledge_base 入库 42 块；get_context 检索返回上下文，RAG 通

## 踩坑：faiss 在 Windows 中文路径下读写崩溃
- 现象：首次 build_knowledge_base 写 .vector_store/index.faiss 报
  RuntimeError: could not open ... for writing: No such file or directory
- 根因：faiss C++ 层用窄字符 fopen，UTF-8 中文路径（如 E:\宋帅\AI输出\...）被认错成"目录不存在"
- 验证：ASCII 临时路径 write_index OK，中文路径 FAIL；目录其实已建好（mkdir 成功，崩在下一行 faiss 打开文件）
- 修复：rag/store.py 的 _save/_load 改为「先写/拷到 ASCII 临时文件再移入存储目录」
- 影响：仅 Windows 中文路径开发机；Linux 部署路径无中文，不受影响
- 顺带：embedder.py 把 BGE 路径硬编码到 .model_cache/models/BAAI--bge-small-zh-v1.5/snapshots/master，
  与 ModelScope snapshot_download(cache_dir=.model_cache) 实际布局一致，下载即匹配，无需改
