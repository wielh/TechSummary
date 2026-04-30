# Partial Index

## 簡介

Partial Index 不是整張表都建索引，只索引「你真的會查的那一部分資料」。

## 語法範例

語法與一般的創建索引類似，關鍵差異就在 WHERE condition

```sql
CREATE INDEX index_name
ON table_name (column1, column2, ...)
WHERE condition;
```

## 優點

1. **減少儲存空間**：索引只包含部分資料，因此索引檔案（.ibd 或 .rel）會顯著縮小。
2. **加速寫入與更新**：如果異動的資料不符合 `WHERE` 條件，資料庫就不需要更新索引。
3. **提高緩存效率**：較小的索引更容易被載入記憶體（Buffer Pool），提升查詢效率。

## 常見應用場景

### 1. 處理「軟刪除 (Soft Delete)」

這可能是最常見的用法。如果你的資料表有 `is_deleted` 或 `deleted_at` 欄位，你通常只會查「還沒被刪除」的資料。

```sql
CREATE INDEX idx_active_users ON users (email) 
WHERE deleted_at IS NULL;
```

### 2. 局部唯一性約束 (Partial Unique Constraint)

有時候你希望某個值在「特定條件下」是唯一的。例如：一個使用者可以有多個過期的訂閱，但只能有一個「使用中」的訂閱。

```sql
CREATE UNIQUE INDEX idx_unique_active_subscription 
ON subscriptions (user_id) 
WHERE status = 'active';
```

### 3. 排除不感興趣的值 (Excluding Common Values)

如果某個欄位有 90% 的資料都是 `NULL` 或某個預設值，而你幾乎不會查這些值，可以將其排除。

```sql
CREATE INDEX idx_important_tasks ON tasks (priority) 
WHERE priority IS NOT NULL AND status != 'done';
```

## 使用注意事項 (CAUTION)

- **查詢條件必須匹配**：
  查詢語句中的 `WHERE` 條件必須包含或符合索引定義的 `WHERE` 條件，查詢優化器（Planner）才會決定使用該 Partial Index。
  
  *範例：*
  - 索引定義：`WHERE status = 'active'`
  - 查詢 1：`WHERE status = 'active'` (會用到索引)
  - 查詢 2：`WHERE status = 'inactive'` (不會用到索引)
  - 查詢 3：不帶 `WHERE` 條件 (不會用到索引)

- **動態參數限制**：
  如果你使用參數化查詢（如 `WHERE status = ?`），優化器可能無法預先知道參數值是否符合索引條件，有時會導致無法使用 Partial Index（視資料庫版本與優化器實作而定）。
