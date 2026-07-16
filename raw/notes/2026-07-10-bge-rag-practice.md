# BGE Embedding + FAISS 向量 RAG 实战

## 演进过程

从 TF-IDF → n-gram 哈希 Embedding → BGE 中文语义 Embedding，最终方案是主流向量 RAG。

## 踩坑清单

### HuggingFace 下载极慢
国内服务器访问 huggingface.co 被墙，下载速度 <20KB/s。尝试了：
- HF-Mirror（hf-mirror.com）：不稳定，有时也慢
- ModelScope（modelscope.cn）：稳定 ~10MB/s，推荐

### ChromaDB 自动下载 ONNX 模型
ChromaDB 的 DefaultEmbeddingFunction 在首次使用时自动下载 all-MiniLM-L6-v2 模型（79MB），导致启动卡住。解决方案：拆掉 ChromaDB，只用 FAISS 做向量库 + JSON 做元数据存储。

### WeChatCrypto 参数名混乱
wechatpy 1.8.18 的 WeChatCrypto 参数名在不同版本中变化：
- 旧版文档写 corpid
- 新版构造器签名写 app_id
- 最终验证：app_id 有效

### WeChatCrypto 缺少 decrypt_echostr 方法
URL 验证时需要手动处理：
1. WeChatSigner(Token, timestamp, nonce, echostr).signature 验证签名
2. PrpCrypto(crypto.key).decrypt(echostr, corp_id) 解密

## 最终技术栈

- 向量库：FAISS IndexFlatIP（余弦相似度）
- Embedding：BAAI/bge-small-zh-v1.5（512 维）
- 元数据：JSON 文件持久化
- 模型下载：ModelScope
- Python 包：sentence-transformers + faiss-cpu + numpy