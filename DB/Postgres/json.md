# PostgreSQL JSON 與 JSONB 使用指南

PostgreSQL 提供兩種儲存 JSON 資料的類型：`json` 和 `jsonb`。這兩者主要的差異在於儲存方式與處理效能。

## 1. `json` vs `jsonb`

| 特性           | `json`                    | `jsonb`                              |
| :------------- | :------------------------ | :----------------------------------- |
| **儲存方式**   | 儲存原始文本的精確副本    | 儲存為分解後的二進位格式             |
| **寫入速度**   | 較快 (無需解析)           | 較慢 (需要轉換為二進位)              |
| **讀取速度**   | 較慢 (每次都需要解析)     | 較快 (預解析格式)                    |
| **空白與順序** | 保留空白與 key 的原始順序 | 不保留空白，會重新排序 key           |
| **索引**       | 不支援 GIN 索引           | **支援 GIN 索引** (極大提升查詢效能) |

> **建議：** 除非有特殊需求（如必須保留原始格式或 key 順序），否則絕大多數情況應優先使用 **`jsonb`**。

---

## 2. 常用操作符 (Operators)

假設有一個資料表 `users`，欄位 `data` 是 `jsonb` 類型，內容如下：
`{"id": 1, "name": "Alice", "tags": ["admin", "staff"], "info": {"age": 25, "city": "Taipei"}}`

| 操作符 | 說明                                        | 範例                    | 結果       |
| :----- | :------------------------------------------ | :---------------------- | :--------- |
| `->`   | 取得 JSON 物件欄位或陣列元素 (回傳 `jsonb`) | `data->'name'`          | `"Alice"`  |
| `->>`  | 取得 JSON 物件欄位或陣列元素 (回傳 `text`)  | `data->>'name'`         | `Alice`    |
| `#>`   | 依路徑取得 JSON 物件 (回傳 `jsonb`)         | `data#>'{info, city}'`  | `"Taipei"` |
| `#>>`  | 依路徑取得 JSON 物件 (回傳 `text`)          | `data#>>'{info, city}'` | `Taipei`   |

---

## 3. 查詢與搜尋 (Search & Containment)

`jsonb` 特有的強大功能是支援包含運算。

- **包含 (`@>`)**：檢查左側 JSON 是否包含右側內容。

  ```sql
  SELECT * FROM users WHERE data @> '{"name": "Alice"}';
  ```

- **被包含 (`<@`)**：檢查右側是否包含左側內容。
- **是否存在鍵值 (`?`)**：檢查字串是否存在於 JSON 的 top-level key 中。
  
  ```sql
  SELECT * FROM users WHERE data ? 'tags';
  ```

---

## 4. 修改 JSONB 資料

### 更新欄位內容 `jsonb_set`

```sql
UPDATE users 
SET data = jsonb_set(data, '{info, age}', '26') 
WHERE id = 1;
```

### 合併 JSON (`||`)

```sql
UPDATE users 
SET data = data || '{"last_login": "2023-10-01"}'::jsonb 
WHERE id = 1;
```

### 刪除欄位 (`-`)

```sql
UPDATE users SET data = data - 'tags' WHERE id = 1;
```

---

## 5. 索引優化 (Indexing)

使用 `jsonb` 最強大的理由之一是可以建立 **GIN (Generalized Inverted Index)** 索引。

### 建立全欄位 GIN 索引

這會對 JSON 內的所有 key 和 value 建立索引，支援 `@>`, `?`, `?&`, `?|` 操作。

```sql
CREATE INDEX idx_users_data ON users USING GIN (data);
```

### 針對特定路徑建立索引 (jsonb_path_ops)

通常比一般的 GIN 索引更小且更快，但僅支援 `@>` 操作。

```sql
CREATE INDEX idx_users_data_path ON users USING GIN (data jsonb_path_ops);
```

---

## 7. 設計建議：一般欄位 vs. JSONB

雖然 `jsonb` 非常強大，但並不代表應該將所有資料都塞進去。以下是設計時的建議判斷基準：

### 適合放入一般欄位 (Regular Columns)

1. **關聯鍵 (Keys)**：主鍵 (Primary Key)、外鍵 (Foreign Key) 必須是獨立欄位。
2. **頻繁過濾與排序**：如果你經常根據某個欄位進行 `WHERE` 過濾或 `ORDER BY` 排序，一般欄位的效能會優於 JSONB。
3. **嚴格的資料校驗**：需要利用資料庫層級的 `NOT NULL`、`CHECK` 約束或資料型別強制的欄位。
4. **固定結構**：所有資料列都擁有的屬性（如：`created_at`, `status`, `user_id`）。
5. **需要進行 JOIN**：作為 Join 條件的欄位必須是一般欄位。

### 適合放入 JSONB 欄位

1. **高度動態的屬性**：屬性種類極多且不固定。例如：電商產品的規格（手機有螢幕尺寸，衣服有材質與尺碼）。
2. **第三方 API 資料**：儲存外部串接的原始回應內容，避免因為對方格式微調而需要頻繁修改 Schema。
3. **稀疏資料 (Sparse Data)**：如果某個屬性只有 1% 的資料會用到，建立獨立欄位會造成大量 `NULL`，使用 JSONB 較省空間且易於維護。
4. **開發初期的彈性**：在需求尚未定型、Schema 頻繁變動的開發初期，可以先用 JSONB 快速迭代，等結構穩定後再抽離出一般欄位。
5. **使用者自訂設定**：如 UI 偏好設定、通知開關等不需要與其他表進行複雜關聯的資料。
