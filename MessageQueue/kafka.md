# kafka  特點介紹

## 高吞吐量

+ 原因 (zero-copy): zero Copy（零拷貝） 是一種 提升資料傳輸效率 的技術，主要用於資料從磁碟或網路搬運到用戶空間（User Space）或傳送到其他裝置的過程中，避免資料在核心空間與使用者空間間的重複拷貝

+ 核心空間: OS 內部運作的區域，具有最高權限。控制硬體（磁碟、網卡、記憶體等）與資源調度, 並提供系統呼叫（System Call）給使用者空間使用。

+ 使用者空間: 運行一般應用程式（如 Nginx、MySQL、VSCode）。權限受限，不能直接存取硬體或記憶體位址。 只能透過系統呼叫向 kernel 請求服務。

+ 以從磁碟讀取資料並透過網路送出為例，傳統資料流如下：
    + 磁碟 → OS 核心空間
    + 核心空間 → 使用者空間（read）
    + 使用者空間 → 核心空間（write）
    + 核心空間 → 網路卡（NIC）

+ 以 linux 為例，linux 是使用 sendfile() 直接傳輸資料，不用將資料複製至使用者空間

## 支持 Exactly Once Semantics 

+ Kafka 0.11+ 預設支援 EOS，只需設定 enable.idempotence=true。kafka 為每個 producer 指派一個 PID（producer ID），每條訊息會帶有 序列號（sequence number），Broker 會記住最後成功的序列號，忽略重複訊息（即使重試）

## Transactional Producer（事務性 Producer）

+ Kafka 支援將多條消息包成一個原子事務提交，以 go 語言為例: 

    ```
    producer.initTransactions()
    producer.beginTransaction()

    producer.send(record1)
    producer.send(record2)

    producer.commitTransaction()
    ```

## Partition 層級順序

+ kafka 的消息是在 Partition 遵循 FIFO 的

## kafka 的限制

+ 原生的kafka不支持延遲消息與重試機制 