# mysql 指令應避免事項

## 索引失效

+ where 的 index 覆蓋率低或查詢沒用到 index

+ 使用 != 、 <>、NOT 查詢可能會變全掃

```sql
SELECT * FROM user WHERE age != 20 ( bad 全掃 )
SELECT * FROM user WHERE age +1 <> 20 ( bad 全掃 )
SELECT * FROM user WHERE age NOT IN(20) ( bad 全掃 )
```

+ 用 like 且 % 在前面有索引也會變全掃

```sql
SELECT * FROM user WHERE name like '%-Mark' ( bad 全掃 )
```

+ 誤用 or 時會變全掃
  + 索引欄位: {age}

    ```sql
    SELECT * FROM user WHERE age = 18 OR name = 'C-Ian'; ( bad 全掃 )
    ```

  + 索引欄位: {age},{name}

    ```sql
    SELECT * FROM user WHERE age = 18 OR name = 'C-Ian'; ( good 索引 )
    ```

  + 索引欄位: {age}

    ```sql
    SELECT * FROM user WHERE age = 18 AND name = 'C-Ian'; ( good 索引 )
    ```

+ 在 WHERE 欄位進行運算
  + 索引欄位: {age}

    ```sql
    SELECT * FROM user WHERE age/2 = 18; ( bad 全掃 )
    SELECT * FROM user WHERE age = 18*2; ( good 索引 )
    ```

+ 使用一些函數

   ```sql
    SELECT * FROM test.user where age >= RAND(); ( bad 全掃 )
   ```

+ 對索引隱式型別轉換

+ 聯合索引非最左匹配

    Note. 有一個比較特殊的查詢條件：where a = 1 and c = 3 ，符合最左匹配嗎？

    MySQL 5.5 的話，前面 a 會走索引，在聯合索引找到主鍵值後，開始回表，到主鍵索引讀取數據行，Server 層從存儲引擎層獲取到數據行後，然後在 Server 層再比對 c 欄位的值。從 MySQL 5.6 之後，有一個索引下推功能，可以在存儲引擎層進行索引遍歷過程中，對索引中包含的欄位先做判斷，直接過濾掉不滿足條件的記錄，再返還給 Server 層，從而減少回表次數。

    索引下推的大概原理是：截斷的欄位不會在 Server 層進行條件判斷，而是會被下推到「存儲引擎層」進行條件判斷（因為 c 欄位的值是在 (a, b, c) 聯合索引裡的），然後過濾出符合條件的數據後再返回給 Server 層。由於在引擎層就過濾掉大量的數據，無需再回表讀取數據來進行判斷，減少回表次數，從而提升了性能。

## 查詢設計不當

### 1. N+1 查詢問題

+ **問題描述**：通常發生在使用 ORM 時，先查詢出一條主資料（1 次），再根據主資料的 ID 在迴圈中查詢相關的子資料（N 次）。這會導致資料庫連線次數過多，造成極大的效能浪費。

+ **案例**：

    ```go
    // 錯誤範例：在迴圈中查詢資料庫
    rows, _ := db.Query("SELECT id FROM orders") // 1 次查詢
    for rows.Next() {
        var orderID int
        rows.Scan(&orderID)
        db.Query("SELECT * FROM items WHERE order_id = ?", orderID) // N 次查詢
    }
    ```

+ **解決方案**：
  + **JOIN**：一次查詢出所有關連資料。
  + **IN 子句**：先收集所有 ID，一次性查詢子資料。

### 2. 深分頁問題 (Deep Paging)

+ **問題描述**：當執行 `LIMIT 1000000, 10` 時，MySQL 並非直接跳到第 100 萬筆。
+ **B+ Tree 底層原理**：
  + **無法直接定位**：B+ Tree 的節點並不儲存「子樹大小」，因此無法像陣列一樣透過索引偏移量直接算出第 100 萬筆的物理位置。
  + **鏈結串列遍歷**：資料庫必須從索引的第一筆開始，沿著葉子節點的「雙向鏈結串列」一筆一筆向後數，數到第 100 萬筆為止。這是一個 $O(N)$ 的操作。
  + **冤枉路 (回表)**：如果使用 `SELECT *`，資料庫在「數數」的過程中，每一筆都會執行磁碟 I/O 去抓取完整資料，最後再丟棄，這才是導致極慢的主因。

+ **解決方案**：
  + **延遲關聯 (Deferred Join)**：
    + **原理**：先在「索引」上完成分頁（只查 ID，不回表），找到那 10 筆 ID 後，再關聯回原表抓取其他欄位。
    + **範例**：

      ```sql
      -- 原始寫法 (慢)
      SELECT * FROM orders WHERE status = 'paid' LIMIT 1000000, 10;

      -- 優化寫法 (快)
      SELECT o.* FROM orders o
      INNER JOIN (
          SELECT id FROM orders WHERE status = 'paid' LIMIT 1000000, 10
      ) AS tmp ON o.id = tmp.id;
      ```

  + **書籤法 (Seek Method / Keyset Pagination)**：
    + **原理**：捨棄 `OFFSET`，改用「上一頁最後一筆 ID」作為過濾條件，讓資料庫直接跳到目標位置。
    + **範例**：

      ```sql
      -- 記住上一頁最後一個 ID 是 1000500
      SELECT * FROM orders WHERE id > 1000500 LIMIT 10;
      ```

    + **缺點**：無法直接跳到特定頁數（例如：跳到第 500 頁），只適合「下一頁」式的翻頁（如手機 APP 的無限捲動）。

### 3. 濫用 `SELECT *`

+ **問題描述**：取出不需要的欄位會增加網路傳輸（I/O）與記憶體開銷，且會導致覆蓋索引 (Covering Index) 失效。

+ **解決方案**：明確指定需要的欄位。

## 全表鎖定

+ 觸發時機
  + DDL 操作:ALTER ,RENAME ,DROP ,CREATE INDEX
  + MyISAM 引擎
  + 優化查詢條件沒有用上索引
