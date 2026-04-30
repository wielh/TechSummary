# Goose - 資料庫遷移工具 (Database Migrations)

[Goose](https://github.com/pressly/goose) 是一個用 Go 編寫的資料庫遷移工具。它支援 SQL 檔案遷移，也支援使用 Go 語言編寫複雜的遷移邏輯。

## 1. 核心特性

- **支援多種資料庫**：PostgreSQL, MySQL, SQLite3, MSSQL, Redshift, ClickHouse 等。
- **雙格式支援**：可以使用純 `.sql` 檔案，也可以使用 `.go` 檔案編寫遷移邏輯。
- **版本控制**：透過一個名為 `goose_db_version` 的資料表來追蹤目前的資料庫版本。
- **簡單易用**：輕量級的 CLI 工具，整合方便。

---

## 2. 安裝方式

你可以透過 Go 安裝 goose CLI：

```bash
go install github.com/pressly/goose/v3/cmd/goose@latest
```

---

## 3. 基本操作語令

| 指令                      | 說明                                     |
| :------------------------ | :--------------------------------------- |
| `goose create <name> sql` | 建立一個新的 SQL 遷移檔案                |
| `goose up`                | 執行所有尚未執行的遷移（升級到最新版本） |
| `goose down`              | 回退上一次執行的遷移                     |
| `goose status`            | 查看目前的遷移狀態                       |
| `goose redo`              | 回退並重新執行最後一次遷移               |
| `goose version`           | 查看目前的資料庫版本號                   |
| `goose fix`               | 將時間戳記格式的檔案重新命名為連續序號   |

---

## 4. 遷移檔案格式 (SQL)

Goose 使用特殊的註解來區分「升級」與「回退」的內容。

### 範例：`20231027100000_create_users_table.sql`

```sql
-- +goose Up
-- +goose StatementBegin
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
-- +goose StatementEnd

-- +goose Down
-- +goose StatementBegin
DROP TABLE users;
-- +goose StatementEnd
```

> **注意：** `-- +goose StatementBegin` 和 `StatementEnd` 用於處理包含多個語句或包含分號（如 Function/Trigger）的情況。

---

## 5. Go 語言遷移檔案

當 SQL 無法滿足複雜邏輯（例如：需要調用外部 API 或進行複雜的資料格式轉換）時，可以使用 Go 檔案。

```go
package migrations

import (
    "context"
    "database/sql"
    "github.com/pressly/goose/v3"
)

func init() {
    goose.AddMigrationContext(UpUsers, DownUsers)
}

func UpUsers(ctx context.Context, tx *sql.Tx) error {
    _, err := tx.ExecContext(ctx, "CREATE TABLE users (id int);")
    return err
}

func DownUsers(ctx context.Context, tx *sql.Tx) error {
    _, err := tx.ExecContext(ctx, "DROP TABLE users;")
    return err
}
```

---

## 6. 在程式碼中整合 Goose

除了 CLI 之外，你也可以在應用程式啟動時自動執行遷移：

```go
import (
    "embed"
    "github.com/pressly/goose/v3"
)

//go:embed migrations/*.sql
var embedMigrations embed.FS

func RunMigrations(db *sql.DB) error {
    goose.SetBaseFS(embedMigrations)

    if err := goose.SetDialect("postgres"); err != nil {
        return err
    }

    if err := goose.Up(db, "migrations"); err != nil {
        return err
    }
    return nil
}
```

---

## 7. Hybrid Versioning (混合版本控制)

在多人開發環境中，版本衝突是常見問題。Goose 推薦使用 **Hybrid Versioning** 流程：

### 核心概念

1. **開發階段使用時間戳記**：
   開發者 A 和 B 同時建立遷移。如果使用序號（005, 006），兩人可能會撞號。使用時間戳記（20231027...）可以有效避免衝突。
2. **生產階段使用連續序號**：
   為了確保執行順序的絕對嚴謹，在合併到主分支後，可以使用 `goose fix` 指令。

### 如何實作

1. **開發**：使用預設的 `goose create` (產生時間戳記檔案)。
2. **整理**：在 CI/CD 流程或合併前，執行 `goose fix`。這會將目錄下所有時間戳記檔案重新命名為 `0001`, `0002`... 等連續序號格式。
3. **部署**：將整理後的序號檔案部署到生產環境。

---

## 8. 最佳實踐

1. **檔案命名**：強烈建議使用時間戳記（預設）而非流水號，避免在多人開發時發生衝突。
2. **不可變性**：一旦遷移檔案已經合併並在正式環境執行，**絕對不要**修改該檔案。應建立一個新的遷移來進行修正。
3. **回退測試**：每次建立 `Up` 遷移時，務必確保 `Down` 遷移也能正確運作。
