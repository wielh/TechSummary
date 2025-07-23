# RockectMQ 特點介紹

## 高吞吐量

+ 原因 (zero-copy): zero Copy（零拷貝） 是一種 提升資料傳輸效率 的技術，主要用於資料從磁碟或網路搬運到用戶空間（User Space）或傳送到其他裝置的過程中，避免資料在核心空間與使用者空間間的重複拷貝

+ rokcetMQ 使用 mmap 實現 zero Copy。資料複製次數比 kafka 多一次，但是 mmap 函數可以獲取發送的消息內容。

## 資料 crud

+ 相比於 kafka是直接將消息寫至partition，rocketmq 的 partition(或者稱為queue) 只是寫了消息的 index 或一些重要資訊。

+ 讀的時候可能會因為需要類似db的回表的操作所以可能會慢一些。

+ 寫的時候同一個 topic 內都是採用單文件(commitlog)，能保證文件的順序寫(append)，從而有潛在的效能提升。

+ 如果需要備份，直接備份單一commitlog即可

## 不支持 Exactly Once Semantics 

+ RocketMQ 在架構上 容許訊息重送，例如：消費者處理失敗，broker 會重新投遞，進而導致不支持 EOS。可能需要由生產者自動產生 messageID 之類的欄位或依賴消費者的處理邏輯實現 EOS 

## 支持死信隊列

+ API 如果消費某消息失敗，可以考慮將其放入此 topic 對應的死信隊列後進行後續處理

## 支持延遲消息

+ rocketMQ 支持消息可以過去指定時間後再取用。