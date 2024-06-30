# 高併發範例一: 搶購

## 基本購買流程

+ 確認用戶訊息

+ 確認訂單資訊與庫存足夠

+ 執行付款，生成訂單

## 高併發解決方案範例

+ 通用:

    + 給資料庫分配更多資源

    + 加上緩存，緩存內容:

        + 活動資訊

        + 用戶名-狀態(處裡中、身分驗證失敗、訂單生成失敗、扣款失敗、成功、活動已關閉/未開始等等)

            + 假設用戶名 int64，狀態為 short，那麼一億個用戶數據為 1GB 左右

            + 如果在緩存上找得到用戶名，則表示已參加過，則可以直接返回

    + 可以多線程併發驗證用戶信息與訂單訊息

    + 驗證完畢後考慮送進 message queue 或程式語言提供的線程安全陣列
    
    + 由另一個函數執行訂單插入，訂單資訊插入資料庫使用 BatchInsert 與 PreparedStatement

    + 監控伺服器資源，如果資源將耗盡應直接限流

+ 如果實時性要求不高，可以用異步處理

    + 先同步執行身分驗證與訂單生成資訊，成功的話返回狀態: 訂單生成成功，然後將此資訊返回客戶端。

    + 後臺起一些執行緒，共用的 thread-saft Array，用來儲存訂單號

        + 假設訂單號為 int64，一億筆會占用 800MB

        + 取出的訂單號須按照時間排序
      
    + 這些執行緒會取用訂單號執行交易(轉帳，扣除庫存，訂單改為OK)。

        + 如果任一步驟失敗，則其他步驟全部取消

        + 如果失敗原因是因為庫存不足，則直接中止動作，不用執行剩下的訂單了。

        + 跑過一遍訂單後，如果還是有剩庫存，可以考慮進行重試