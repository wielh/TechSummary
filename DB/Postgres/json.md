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

## 2. 設計建議：一般欄位 vs. JSONB

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

---

## 3. 索引優化 (Indexing)

使用 `jsonb` 最強大的理由之一是可以建立 **GIN (Generalized Inverted Index)** 索引。

### 3.1 GIN 索引原理 (How GIN Works)

GIN 代表 **Generalized Inverted Index**（廣義倒排索引）。它特別適合用來處理像 JSONB 這種可以被拆解成多個鍵值對 (key-value pairs) 或陣列元素的複雜資料類型。

- **倒排索引機制**：
  傳統 B-Tree 索引是將整個欄位值對應到資料列。而 GIN 是將 JSONB 內部的每一個 **Key** 和 **Value** 提取出來，分別建立索引項目。每個索引項目會記錄一份清單，指向包含該項目的一系列資料列編號 (TIDs)。
  
- **查詢流程**：
  當執行 `@>` (包含) 查詢時，PostgreSQL 會：
  1. 將查詢條件拆解成多個鍵/值項目。
  2. 在 GIN 索引中查找這些項目對應的資料列清單。
  3. 取這些清單的 **交集 (Intersection)**，快速定位出符合所有條件的資料列。
  
- **Bitmap Scan**：
  GIN 查詢通常會產生一個 Bitmap，記錄哪些頁面包含匹配的資料，這能有效減少隨機 I/O 讀取，並提升大範圍搜尋的效能。

### 3.2 GIN 索引類型

#### A. 預設 GIN 索引 (`jsonb_ops`)

這是建立 GIN 索引時的預設運算子類別。它會為 JSONB 中的每個 key、value 和陣列元素建立索引。

- **建立語法：**

  ```sql
  CREATE INDEX idx_users_data ON users USING GIN (data);
  ```

- **支援的操作符：** `@>`, `?`, `?&`, `?|`。
- **優點：** 彈性最高，支援多種搜尋方式。
- **缺點：** 索引體積較大。

#### B. 路徑 GIN 索引 (`jsonb_path_ops`)

這種類型的索引只會為 JSONB 內部的「路徑 + 值」建立雜湊 (Hash) 索引。

- **建立語法：**

  ```sql
  CREATE INDEX idx_users_data_path ON users USING GIN (data jsonb_path_ops);
  ```

- **支援的操作符：** 僅支援包含操作符 `@>`。
- **優點：** 體積明顯比預設 GIN 小（通常只有一半），且對於包含搜尋 (`@>`) 的效能更好。
- **缺點：** 彈性較低，不支援 `?` 等檢查 Key 是否存在的操作。

### 3.3 運算子類別比較表

| 索引類別     | 關鍵字           | 支援操作符          | 索引大小 | 效能 (Containment) |
| :----------- | :--------------- | :------------------ | :------- | :----------------- |
| **預設 GIN** | `jsonb_ops`      | `@>`, `?`, `?&`, `? | `        | 較大               | 一般 |
| **路徑 GIN** | `jsonb_path_ops` | `@>`                | 較小     | 較優               |

---

### 3.4 運算式索引 (Expression Index / Functional Index)

如果你知道應用程式最常用來查詢的是 JSONB 內的某個特定欄位，且該欄位的值是簡單的標量（如字串、數字），則可以使用一般的 **B-Tree 索引**。這比 GIN 索引更輕量、速度更快。

- **建立語法：**

  ```sql
  -- 必須使用括號將運算式括起來
  CREATE INDEX idx_users_name ON users ((data->>'name'));
  ```

- **使用情境：**

  ```sql
  -- 此查詢將會使用上述 B-Tree 索引
  SELECT * FROM users WHERE data->>'name' = 'Alice';
  ```

- **優點：** 體積最小，支援排序 (`ORDER BY`) 與範圍查詢 (`>`, `<`)。
- **限制：** 只能針對這一個特定路徑加速，無法應付動態的 JSON 結構搜尋。

---

## 4. 常用操作符 (Operators)

假設有一個資料表 `users`，欄位 `data` 是 `jsonb` 類型，內容如下：
`{"id": 1, "name": "Alice", "tags": ["admin", "staff"], "info": {"age": 25, "city": "Taipei"}}`

| 操作符 | 說明                                        | 範例                    | 結果       |
| :----- | :------------------------------------------ | :---------------------- | :--------- |
| `->`   | 取得 JSON 物件欄位或陣列元素 (回傳 `jsonb`) | `data->'name'`          | `"Alice"`  |
| `->>`  | 取得 JSON 物件欄位或陣列元素 (回傳 `text`)  | `data->>'name'`         | `Alice`    |
| `#>`   | 依路徑取得 JSON 物件 (回傳 `jsonb`)         | `data#>'{info, city}'`  | `"Taipei"` |
| `#>>`  | 依路徑取得 JSON 物件 (回傳 `text`)          | `data#>>'{info, city}'` | `Taipei`   |

---

## 5. 查詢與搜尋 (Search & Containment)

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

## 6. 修改 JSONB 資料

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
