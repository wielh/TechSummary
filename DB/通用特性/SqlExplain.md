# SQL 效能優化與 EXPLAIN 執行計劃分析

在資料庫開發中，當遇到查詢緩慢、CPU 飆高時，最核心的診斷工具就是 **`EXPLAIN`**。它能讓資料庫告訴我們它準備「如何」執行這條 SQL 語句，包括是否使用了索引、以何種方式掃描資料表、以及多表 Join 的順序。

---

## 1. 什麼是 `EXPLAIN`？

我們只需在任何 `SELECT`、`UPDATE`、`DELETE` 或 `INSERT` 語句前加上 `EXPLAIN` 關鍵字，即可獲取其執行計劃（Execution Plan）：

```sql
EXPLAIN SELECT * FROM users WHERE age > 20;
```

其輸出會包含一組表格，描述了查詢的步驟與對應的資源開銷估計。

---

## 2. EXPLAIN 核心輸出欄位詳解

| 欄位名 | 作用 | 說明 |
| :--- | :--- | :--- |
| **`id`** | 查詢的序號 | 描述多表查詢的執行順序。id 越大的步驟越先執行；id 相同則從上到下執行。 |
| **`select_type`** | 查詢的類型 | 常見有 `SIMPLE`（簡單查詢）、`PRIMARY`（最外層主查詢）、`SUBQUERY`（子查詢）等。 |
| **`table`** | 正在存取的資料表 | 顯示該步驟是在讀取哪一張表（或衍生出來的暫存表 `derived`）。 |
| **`type`** | **訪問類型 (Access Type)** | **最關鍵的效能指標之一**，表示資料庫如何尋找資料列（詳見下述）。 |
| **`possible_keys`** | 可能用上的索引 | 資料庫在分析時，發現哪些索引「有可能」幫上忙。 |
| **`key`** | **實際採用的索引** | 如果為 `NULL`，代表此查詢發生了**全表掃描 (Full Table Scan)**，需要優化。 |
| **`key_len`** | 實際使用索引的長度 | 以位元組（Bytes）為單位。可用來判斷「聯合索引」中到底有多少個欄位被實際用上。 |
| **`rows`** | 預估掃描的行數 | 資料庫估計需要掃描多少行才能找到目標數據。這個數字越小越好。 |
| **`filtered`** | 過濾百分比 | 經由 WHERE 條件過濾後剩餘資料的估算佔比，數值越高代表索引過濾效果越精準。 |
| **`Extra`** | **額外附加資訊** | **極為關鍵的效能指標之二**，包含了是否使用暫存表、是否排序等細節。 |

---

## 3. `type` 訪問類型層級（效能由優到劣）

`type` 欄位是評估查詢效率最直觀的依據。我們優化的目標是**盡可能將 type 提升到 `range` 以上**，避免 `index` 和 `ALL`。

1. **`system`**：表只有一行資料（系統表）。這是最理想的情況，但一般業務開發中極少遇到。
2. **`const`**：透過**主鍵 (Primary Key)** 或**唯一索引 (Unique Index)** 進行等值查詢，只會找到一筆記錄。極快。
   * 範例：`SELECT * FROM users WHERE id = 1;`
3. **`eq_ref`**：多表 Join 時，關聯條件使用的是被驅動表的**主鍵或唯一索引**。每一條記錄都只會對應到另一張表的一行。
   * 範例：`SELECT * FROM a JOIN b ON a.id = b.user_id;`（若 `b.user_id` 是主鍵或唯一鍵）
4. **`ref`**：透過**非唯一索引（一般單列索引或聯合索引的前綴）**進行等值查詢，可能會匹配到多行資料。
   * 範例：`SELECT * FROM users WHERE name = 'Alice';`（`name` 為一般索引）
5. **`range`**：利用索引進行**範圍掃描**（通常包含 `>`, `<`, `BETWEEN`, `IN`, `LIKE 'A%'` 等操作符）。
   * 範例：`SELECT * FROM users WHERE age BETWEEN 10 AND 20;`
6. **`index`**：**全索引樹掃描 (Full Index Scan)**。雖然也是掃描整張表，但因為它直接在記憶體中掃描「索引樹」，而不需要去磁碟讀取完整的資料行，因此依然比 `ALL` 快。
   * 範例：`SELECT id FROM users;`（因為只需查 `id`，直接掃描主鍵索引樹即可，不需回表）
7. **`ALL`**：**全表掃描 (Full Table Scan)**。資料庫必須從頭到尾掃描磁碟上的整張資料表，在資料量大時會造成災難性的 I/O 阻塞。

---

## 4. `Extra` 關鍵輔助資訊解讀

`Extra` 欄位會告訴我們很多優化的細節細訊：

* **`Using index` (覆蓋索引，極優)**：
  * **含意**：查詢所需要的欄位全部都在索引樹中，**完全不需要「回表」**去主鍵索引找完整資料。
  * **優化建議**：這是最理想的狀態，可透過設計聯合索引來達成。
* **`Using index condition` (索引下推 ICP，佳)**：
  * **含意**：在聯合索引中，雖然部分欄位無法用來直接定位，但引擎層會先用這些欄位進行資料過濾，減少回表讀取不必要行數的次數。詳細原理可參考 [mysql 指令應避免事項 - 聯合索引非最左匹配](file:///e:/code/TechSummary/DB/通用特性/DBSlow.md#L56-L63)。
* **`Using where` (伺服器層過濾，普通)**：
  * **含意**：儲存引擎層返回資料後，MySQL Server 層還需要使用 WHERE 條件進行二次過濾。這代表索引沒有完全過濾掉所有不符條件的資料。
* **`Using filesort` (檔案排序，警告 ⚠️)**：
  * **含意**：MySQL 無法利用索引完成排序，必須在記憶體或磁碟中將資料拿出來重新進行排序。這是一個**極其耗費 CPU** 的操作。
  * **優化建議**：通常需要為 `ORDER BY` 的欄位與 `WHERE` 條件欄位建立聯合索引。
* **`Using temporary` (臨時表，危險 ⚠️⚠️)**：
  * **含意**：MySQL 在處理查詢時（如 `GROUP BY`, `DISTINCT`, `UNION`），必須建立一張內部的臨時表來暫存中間結果。
  * **優化建議**：效能開銷極大，應透過調整索引結構或改寫 SQL 來避免。

---

## 5. 優化實戰案例

### 案例 A：消除全表掃描 (`ALL` $\to$ `ref`)
* **慢 SQL**：
  ```sql
  EXPLAIN SELECT * FROM orders WHERE status = 'pending';
  -- 輸出結果：type = ALL, key = NULL, rows = 50000
  ```
* **優化方案**：在 `status` 欄位建立索引：
  ```sql
  CREATE INDEX idx_status ON orders(status);
  ```
* **優化後**：
  ```sql
  EXPLAIN SELECT * FROM orders WHERE status = 'pending';
  -- 輸出結果：type = ref, key = idx_status, rows = 120
  ```

### 案例 B：消除檔案排序 (`Using filesort` $\to$ `Using index`)
* **慢 SQL**：
  ```sql
  EXPLAIN SELECT * FROM users WHERE status = 1 ORDER BY created_at DESC;
  -- 輸出結果：type = ref, key = idx_status, Extra = Using filesort
  -- 原因：雖然用了 status 索引，但排序 created_at 卻無法利用該索引，導致 filesort。
  ```
* **優化方案**：建立**聯合索引**，將過濾欄位放在最左邊，排序欄位放在後面：
  ```sql
  CREATE INDEX idx_status_created ON users(status, created_at);
  ```
* **優化後**：
  ```sql
  EXPLAIN SELECT * FROM users WHERE status = 1 ORDER BY created_at DESC;
  -- 輸出結果：type = ref, key = idx_status_created, Extra = Using index condition (已無 filesort)
  ```
