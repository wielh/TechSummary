# GORM Gen - 類型安全的代碼生成工具

[GORM Gen](https://github.com/go-gorm/gen) 是 GORM 官方推出的一個代碼生成工具，旨在提供更安全、更高效的資料庫操作方式。

## 1. 核心特性

- **類型安全 (Type-safe)**：完全避免了使用字串來指定欄位名稱（如 `Where("name = ?", "alice")`），改為使用強類型的物件屬性。
- **編譯檢查**：查詢錯誤可以在編譯時期被發現，而非執行期。
- **自動生成 DAO**：可以從現有的資料庫表或 Go 結構體自動生成查詢代碼。
- **效能優化**：生成的代碼經過優化，執行效率高。
- **支援自定義 SQL**：可以透過介面定義自定義的查詢邏輯並自動生成相應的方法。

---

## 2. 安裝方式

```bash
go get -u gorm.io/gen
```

---

## 3. 基本使用流程

### 第一步：配置生成器 (Generator)

建立一個 `cmd/generate/main.go` 檔案：

```go
package main

import (
    "gorm.io/gen"
    "gorm.io/gorm"
    "gorm.io/driver/postgres"
)

func main() {
    g := gen.NewGenerator(gen.Config{
        OutPath: "../../internal/query", // 代碼生成後的存放路徑
        Mode:    gen.WithoutContext | gen.WithDefaultQuery | gen.WithQueryInterface,
    })

    // 連接資料庫
    db, _ := gorm.Open(postgres.Open("dsn"), &gorm.Config{})
    g.UseDB(db)

    // 為所有表生成代碼
    g.ApplyBasic(g.GenerateAllTable()...)

    // 開始執行生成
    g.Execute()
}
```

### 第二步：使用生成的代碼進行查詢

生成的代碼會放在 `internal/query` 目錄下，你可以像這樣使用：

```go
import "your-project/internal/query"

func GetUser(id int64) {
    u := query.User
    user, err := u.WithContext(ctx).Where(u.ID.Eq(id)).First()
    // ...
}
```

---

## 4. 進階功能：自定義查詢 (DIY Methods)

你可以定義一個介面來描述複雜的 SQL 邏輯：

```go
type Querier interface {
    // SELECT * FROM @@table WHERE name = @name{{if role != ""}} AND role = @role{{end}}
    FilterByNameAndRole(name string, role string) ([]gen.T, error)
}
```

然後在生成器中套用：

```go
g.ApplyInterface(func(Querier) {}, g.GenerateModel("users"))
```

Gen 會自動根據註解中的 SQL 生成對應的 Go 方法。

---

## 5. 進階應用

### 1. 動態修改 GORM Tag (`gen.FieldGORMTag`)

GORM Gen 預設不生成 index 相關資訊，但可以根據資料庫中的索引資訊，動態地為生成的 Go 結構體加上 `gorm:"index:..."` 標籤。

```go
opts := []gen.ModelOpt{
    gen.FieldGORMTag(colName, func(tag string) string {
        if isIndex {
            return tag + ";index"
        }
        return tag
    }),
}
g.GenerateModel(tableName, opts...)
```

### 2. 模擬 Embedded 方法 (`gen.FieldNew` + `gen.FieldIgnore`)

當你需要將多個欄位收納進一個 Embedded 結構體時，可以使用 `gen.FieldNew` 增加該結構體欄位，並用 `gen.FieldIgnore` 排除原始的散落欄位。

```go
opts := []gen.ModelOpt{
    // 建立一個帶有 embedded 標籤的匿名結構體欄位
    gen.FieldNew("", "MaterialResource", field.Tag{}.Set("gorm", "embedded")),
    // 忽略原本已包含在 MaterialResource 內的個別欄位
    gen.FieldIgnore("oid", "id", "product_id", "status"),
}
```

### 3. 指定自定義類別方法 (`gen.FieldType`)

將資料庫欄位映射到特定的 Go 類型（如自定義的 Enum、JSON 結構體或高效能時間類型）。**注意：** 若自定義類別位在其他 package，需配合 `g.WithImportPkgPath` 使用。

```go
// 1. 在生成器配置中指定 import 路徑
g.WithImportPkgPath("your-project/pkg/types", "your-project/internal/workorder")

// 2. 指定欄位映射
opts := []gen.ModelOpt{
    // 將 status 欄位映射到專案自定義的 Enum 類別
    gen.FieldType("status", "workorder.Status"),
    // 將 detail 欄位（通常是 JSONB）映射到特定的結構體
    gen.FieldType("detail", "CollectRecordDetail"),
    // 將時間欄位映射到自定義的奈秒時間類別
    gen.FieldType("created_at", "types.TimeNano"),
}
```

### 4. `gorm.ColumnType` 介面的應用

在生成過程中，你可以透過 `db.Migrator().ColumnTypes(tableName)` 取得實作了 `ColumnType` 介面的物件，進而獲取精確的欄位元資料。

`gorm.ColumnType` 完整介面如下：

```go
type ColumnType interface {
    Name() string
    DatabaseTypeName() string                 // varchar, int8 等
    ColumnType() (columnType string, ok bool) // varchar(64) 等完整型別
    PrimaryKey() (isPrimaryKey bool, ok bool)
    AutoIncrement() (isAutoIncrement bool, ok bool)
    Length() (length int64, ok bool)
    DecimalSize() (precision int64, scale int64, ok bool)
    Nullable() (nullable bool, ok bool)
    Unique() (unique bool, ok bool)
    ScanType() reflect.Type
    Comment() (value string, ok bool)
    DefaultValue() (value string, ok bool)
}
```

這讓你可以撰寫邏輯（例如檢查 `Nullable()` 或 `PrimaryKey()`）來自動判斷該為哪些欄位套用何種 `gen.FieldType` 或 `gen.FieldGORMTag`。

#### 各方法回傳值說明

- **`Name()`**: 返回欄位在資料庫中的名稱（例如 `user_id`）。
- **`DatabaseTypeName()`**: 返回資料庫內部的型別名稱（例如 `VARCHAR`, `BIGINT`, `TIMESTAMPTZ`）。
- **`ColumnType()`**: 返回包含長度的完整型別描述（例如 `VARCHAR(255)`），`ok` 表示是否成功取得。
- **`PrimaryKey()`**: 返回該欄位是否為主鍵（Primary Key）。
- **`AutoIncrement()`**: 返回該欄位是否為自動遞增（Auto Increment）。
- **`Length()`**: 返回欄位的最大長度（適用於字串等類型）。
- **`DecimalSize()`**: 返回數值類型的精度（Precision）與比例（Scale），例如 `DECIMAL(10,2)` 會返回 `10, 2`。
- **`Nullable()`**: 返回該欄位是否允許為 NULL。
- **`Unique()`**: 返回該欄位是否具有唯一約束（Unique Constraint）。
- **`ScanType()`**: 返回 Go 語言中可以用來接收該欄位資料的 `reflect.Type`。
- **`Comment()`**: 返回欄位的註解文字。
- **`DefaultValue()`**: 返回欄位在資料庫中定義的預設值字串。

### 5. 處理資料庫視圖 (View)

GORM Gen 預設主要處理 Table，但你可以透過查詢系統表取得 View 名稱，並將其視為一般 Table 進行生成，這對於複雜的報表查詢非常有用。

---

## 6. 為什麼要使用 GORM Gen？

| 傳統 GORM                     | GORM Gen                      |
| :---------------------------- | :---------------------------- |
| 使用字串指定欄位，易寫錯      | 使用物件屬性，有 IDE 自動補完 |
| 拼寫錯誤只能在運行時發現      | 拼寫錯誤會導致編譯失敗        |
| 需要手動寫大量 CRUD 樣板代碼  | 全自動生成，只需關注核心業務  |
| 複雜查詢容易出現 SQL 注入風險 | 內建參數化查詢，結構更安全    |

---

## 7. 注意事項

1. **代碼同步**：每當資料庫 Schema 變更後，務必重新執行生成指令。
2. **OutPath 路徑**：建議將生成的代碼放在獨立的 package 中（如 `query`），避免與業務邏輯或 Model 混在一起。
3. **版本匹配**：確保 `gorm.io/gen` 與 `gorm.io/gorm` 的版本相容。
