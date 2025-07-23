# 細節

## RAG 注意事項
+  對於不同的資料種類需要不同的分塊策略，可能在建立知識庫的時候心裡既要有預期資料格式應該要長什麼樣。
+  無法應對全局問題(比如一個文章有幾個"關鍵字")，因為返回的僅是文章的一個段落。
+  代名詞與日期問題: 比如 "我" 或 "今天" 之類的詞，可能會因為被截斷而無法判斷。
+  對於比較問題回答得比較不好，可以考慮以下解法
    + 比較 A 與 B => 被 llm 拆成問 A 與 B 的性質
    + 分別在資料庫中搜尋A與B的資料
    + 再返回給 llm，特別指名 llm 列出相異部分。
+  縮寫的表現不好(比如 ACID)，請務必將問題敘述清楚，不要用縮寫。
+  表格切分問題。

## chunking 策略 
+  重疊式分塊（Sliding Window / Overlapping Chunking）: 每個 chunk 重疊一部分內容（設定 chunk_overlap）	保持上下文連貫，對問答效果好	
+  基於結構的分塊（Structure-Aware Chunking）: 根據段落、標題、章節、HTML 元素等自然結構進行分塊
+  基於分隔符的遞迴分塊（Recursive Character Splitter）: 使用多層級分隔符（如段落、句子、空格）遞迴切割直到符合限制	
+  語義分塊（Semantic Chunking）: embedding 後相似的內容合併再一起。
+  llm 分塊: 請 llm 幫助我們分塊。

## RAG 策略
+ simple-RAG: 根據問句與提交從vector_db搜尋而來的前k個結果給llm評判並生成結果。
+ Self-RAG: llm 根據問句與提交從vector_db搜尋而來的前k個結果給llm評判並生成結果。簡單來說，llm自主生成結果，自主判斷答案合理，自主產生新問題。
+ adaptive-RAG: Q1 -> A1, Q1+A1 -> A2, ....

