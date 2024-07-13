# MySQL, Mongodb 比較

## 開發便捷:

+ mysql 必須事先規劃資料格式(schema)，而 mongodb 可以直接將所有資料都直接寫入。但是後續維護資料較為麻煩。

+ mongodb 支援豐富的查詢語言和靈活的資料模型， 適合用在資料格式不確定 (unstable schema)，而未來很有可能調整或快速測試功能。

+ mongodb 沒有強制建立 forgien key的方式，需要由程式層面維護。

## 性能:

+ MySQL：對於具有複雜查詢和交易的應用程序，性能可能更高，特別是當資料庫結構化良好時。

+ MongoDB：對於讀取操作和大規模的讀/寫操作，(因為不用預先處理資料格式)尤其是對於含有大量非結構化資料的應用，可能會提供更快的性能。

+ MySQL 可能因為需要符合範式而將資料分離，而 MongoDB 可將這些資料都放到 Document，I/O性能自然提升

## ACID

+ MySQL 可以在 column 層級達成ACID，而MongoDB 在單文檔操作中是符合 ACID 特性。

+ MongoDB 預設是關閉ACID的，而是採用BASE模式。

+ MongoDB 勝在性能與擴展性，而MySQL則保證交易結果可靠與資料強一致



