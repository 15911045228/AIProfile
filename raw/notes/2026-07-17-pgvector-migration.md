# 知识库迁移到 PostgreSQL pgvector（2026-07-17）

## 背景
之前知识库用 FAISS 本地文件（.vector_store/）存储向量索引，遇到 Windows 中文路径问题。
改用 pgvector 后，向量和业务数据统一在 PostgreSQL 中管理。

## 改造内容
- 新增 `knowledge_chunks` 表，含 `embedding vector(512)` 列
- `rag/store.py` 从 FAISS IndexFlatIP 改为 PostgreSQL `<=>` 余弦距离精确搜索
- `rag/reranker.py` 新增 cross-encoder 重排序（BGE-reranker-v2-m3）
- `rag/engine.py` 检索流程：向量搜索 top 15 → reranker 精排 top 3

## 关键决策
- ivfflat 索引对 42 条数据反而降低召回率（lists=100 时候选稀疏），改用精确搜索
- 精确搜索对 <10K 行的数据量完全够用，全表扫描向量计算在毫秒级
- reranker 模型因 HuggingFace 被墙暂未下载，代码有 fallback（跳过 reranker）

## 数据库架构（6 张表）
- users：微信用户 + JSONB 画像
- conversations：会话 + 游标
- messages：消息记录
- kf_cursors：微信客服游标
- ai_logs：AI 处理日志（模型/耗时/token）
- knowledge_chunks：知识库块 + 向量