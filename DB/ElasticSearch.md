# Elasticsearch 簡介

## 簡介

ElasticSearch（彈性搜尋）：是一款開源的開源、RESTful風格的搜尋和資料分析引擎，它底層基於Apache Lucene開源函式庫進行封裝，其橫跨提供了多個使用者能力的全文搜尋引擎，還可以被準確表述：

+ 一個全方位的即時文檔存儲，每個欄位都可以索引和搜尋；

+ 一個全球即時分析搜尋引擎；

+ 能勝任上百個節點的擴展，並支援PB等級格式化與非格式化數據

2. 名詞對應

|Elasticsearch	|MongoDB	|SQL|
| --------- | --------- |--------- |
|index	|collection	|table
|document	|document	|row
|field	|field	|column
|Inverted_index	|index	|index

3. 實作 inverted_index:

實作一個簡單的 invert index 只有兩個步驟：

取出該文件的文字 (token)，並將這些文字標準化 (e.g 取詞幹，移除停用詞)。
迭代所有文字，並在 invert index 中找出該文字的位置，並將文件的 ID 加入。
而從 invert index 找出對應的文件也只有兩個步驟：

   + 將搜尋條件標準化
   + 迭代所有搜尋條件，並從 invert index 中找出對應的文件 ID 並取聯集。
   
雖然 invert index 的實作流程看似簡單，但卻能大大提升全文字搜尋的精準度以及搜尋速度，此外設計 invert index 的難度在於如何提升建立索引的速度以及正確的從文件中取出文字並標準化。

在實作中，invert index 也可以加上 metadata，例如各個 token 在所有 document 中出現的次數，並以此當作 token 的權重，以及 token 在 document 中的位置，以此來精準定位出 document 中的段落。
