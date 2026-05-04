# PostgreSQL JOIN LATERAL

`JOIN LATERAL` 是 PostgreSQL 中一個強大且靈活的語法，它允許子查詢（Subquery）參考同一層級中出現在它之前的資料表欄位。

你可以將其想像成程式語言中的 **`for-each` 迴圈**：針對左側資料表的每一列，各執行一次右側的子查詢。

---

## 1. 核心差異

### 一般 JOIN (Standard JOIN)

右側的子查詢是「獨立」的，它無法看見左側資料表的任何資訊。

```sql
-- 這會報錯，因為子查詢看不到 t1.id
SELECT * FROM table1 t1, (SELECT * FROM table2 t2 WHERE t2.ref_id = t1.id) ss;
```

### LATERAL JOIN

右側的子查詢可以「橫向 (Lateral)」存取左側資料表的欄位。

```sql
-- 這是合法的
SELECT * FROM table1 t1, LATERAL (SELECT * FROM table2 t2 WHERE t2.ref_id = t1.id) ss;
```

---

## 2. 與一般 JOIN 的深入對比

| 特性         | 一般 JOIN (Standard JOIN)        | LATERAL JOIN                                |
| :----------- | :------------------------------- | :------------------------------------------ |
| **運作原理** | 集合與集合的匹配 (Set Matching)  | 針對每一列執行子查詢 (Row-by-row Execution) |
| **可見度**   | 子查詢內部**看不到**外部表格欄位 | 子查詢內部**可以存取**外部表格欄位          |
| **執行次數** | 通常子查詢只執行一次，再進行合併 | 左表有多少列，子查詢就執行多少次 (邏輯上)   |
| **典型場景** | 簡單的 1:1 或 1:N 關聯           | 分組前 N 名、展開 JSON/陣列、動態計算       |

### 核心差異範例

**想取出每個分類的「分組最高價產品」：**

* **使用一般 JOIN (難以達成)**：你必須先算出所有分類的最高價，再關聯回去，或者使用複雜的 Window Function。
* **使用 LATERAL (直覺簡單)**：

    ```sql
    SELECT c.name, p.name, p.price
    FROM categories c
    JOIN LATERAL (
        SELECT name, price FROM products 
        WHERE category_id = c.id      -- 關鍵：參考外部欄位
        ORDER BY price DESC LIMIT 1   -- 關鍵：直接在子查詢限制
    ) p ON true;
    ```

---

## 3. 常見使用場景

### A. 取得每一組的前 N 名 (Top-N per Group)

這是 `LATERAL` 最經典的用途。例如：取得每個分類中最新上市的 3 個產品。

```sql
SELECT 
    c.category_name, 
    p.product_name, 
    p.price
FROM categories c
LEFT JOIN LATERAL (
    SELECT name AS product_name, price
    FROM products
    WHERE category_id = c.id  -- 參考左表的 c.id
    ORDER BY created_at DESC
    LIMIT 3                   -- 限制每個分類只取 3 筆
) p ON true;
```

### B. 搭配集值函數 (Set-Returning Functions)

當你需要對每一列資料執行 `jsonb_array_elements` 或 `unnest` 等會回傳多列結果的函數時。

```sql
-- 假設 users 表有一個 tags JSONB 欄位: ["golang", "postgres"]
SELECT 
    u.username, 
    t.tag_name
FROM users u
CROSS JOIN LATERAL jsonb_array_elements_text(u.tags) AS t(tag_name);
```

### C. 複雜計算的重用

如果你在 `SELECT` 中需要進行複雜計算，且後續的過濾或計算需要用到這個結果，`LATERAL` 可以避免重複撰寫相同的表達式。

```sql
SELECT 
    p.name, 
    calc.subtotal, 
    calc.tax, 
    calc.total
FROM products p
CROSS JOIN LATERAL (
    SELECT 
        p.price * 10 AS subtotal,
        (p.price * 10) * 0.05 AS tax,
        (p.price * 10) * 1.05 AS total
) calc;
```

---

## 4. 語法小撇步

* `CROSS JOIN LATERAL` 等同於在 `FROM` 後面直接用逗號連接並加上 `LATERAL`（類似於 `INNER JOIN`）。
* `LEFT JOIN LATERAL ... ON true` 則類似於 `LEFT JOIN`，即使子查詢沒有回傳結果，左側的列也會保留。

---

## 5. 總結

* **關鍵字**：`for-each`。
* **何時使用**：當你的子查詢需要用到「前一張表」的欄位值時。
* **優點**：解決了許多以往需要靠 `Window Functions` (如 `ROW_NUMBER()`) 才能解決的複雜分組問題，且語法通常更直覺。
