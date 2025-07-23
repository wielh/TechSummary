# LangChain framework

## 簡介
LangChain 是一個專為構建 具上下文記憶、推理與工具整合能力的 LLM 應用 所設計的 Python 框架，目標是讓開發者快速打造聊天機器人、問答系統、RAG 系統等 AI 應用。

## 核心功能

+ Document Loaders: 載入各類型文件（PDF、TXT、Notion、網頁…）
+ Text Splitter: 將長文切成 Chunk，便於向量化與查詢
+ Embedding: 將文字轉為向量表示，可選 OpenAI、SentenceTransformer…
+ LLMs:	封裝各種大型語言模型（OpenAI, Anthropic, HuggingFace, Ollama...）
+ Prompts:	建立與管理提示詞模板，支援參數化與多輪訊息組合
+ VectorStores:	向量儲存與相似度查詢，如 FAISS、Chroma、PGVector…
+ Retrievers: 根據 query 從向量庫中取出相關文件，可以支援 similarity 與 metadata search

+ Chains:	建構 LLM 運算流程，如 Prompt → LLM → Output → Tool
+ Agents:	可根據指令自動選擇工具執行（具推理能力）
+ Tools:	定義可被 Agent 使用的工具（例如查資料庫、Google 搜尋）
+ Memory:	管理對話歷史與上下文記憶，可長期記錄
+ Output Parsers:	將 LLM 回應轉成特定格式（結構化 JSON 等）