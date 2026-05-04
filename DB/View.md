# Database View (視圖)

View 是一張**「虛擬資料表」**。它本身並不儲存實際的資料內容，而是儲存了一段 **SQL 查詢指令**。當你查詢一個 View 時，資料庫會執行背後的那段 SQL，並將結果像一般資料表一樣呈現。

---

## 1. 核心作用

### A. 簡化複雜查詢 (Simplification)

* **封裝邏輯**：將複雜的 `JOIN`、子查詢、彙總計算（`SUM`, `AVG`）或 `CASE WHEN` 邏輯封裝起來。
* **重用性**：開發者只需要 `SELECT * FROM view_name`，不需要每次重複撰寫冗長的 SQL。

### B. 安全性與存取控制 (Security)

* **隱藏敏感資料**：建立一個排除敏感欄位（如密碼、身分證字號）的 View，只開放此 View 給特定權限的使用者。
* **過濾資料列**：透過 `WHERE` 條件限制使用者只能看到特定範圍的資料（例如：只能看到「啟用中」的訂單）。

### C. 邏輯獨立性 (Decoupling)

* **架構解耦**：當底層表結構發生變動（例如：欄位改名、一張表拆分成兩張）時，只需要修改 View 的 SQL 定義。
* **前端無感**：只要 View 的名稱與輸出欄位不變，應用程式（前端）的程式碼完全不需要改動。

---

## 2. View 的種類

| 種類                             | 說明                                | 優點                 | 缺點                              |
| :------------------------------- | :---------------------------------- | :------------------- | :-------------------------------- |
| **一般視圖 (Standard View)**     | 只儲存 SQL 指令，查詢時才動態執行。 | 不佔空間、資料即時。 | 複雜查詢時效能較低。              |
| **物化視圖 (Materialized View)** | 將查詢結果**實體儲存**在磁碟中。    | 讀取速度極快。       | 資料非即時，需手動/排程重新整理。 |

---

## 3. SQL 範例

### 建立 View

```sql
CREATE VIEW v_active_employee_contacts AS
SELECT 
    e.id,
    e.name AS emp_name,
    d.dept_name,
    e.email
FROM employees e
JOIN departments d ON e.dept_id = d.id
WHERE e.status = 'active';
```

### 查詢 View

```sql
SELECT * FROM v_active_employee_contacts WHERE dept_name = 'Engineering';
```

---

## 4. 使用建議與限制

* **效能考量**：過度嵌套的 View（View 查 View）會導致查詢計畫變得極度複雜，影響效能。
* **唯讀性**：雖然某些簡單的 View 支援 `INSERT/UPDATE`，但包含 `JOIN`、`GROUP BY` 或 `DISTINCT` 的 View 通常是**唯讀**的。
* **命名慣例**：建議加上前綴（如 `v_` 或 `vw_`），以便在查看資料庫結構時快速區分 Table 與 View。
