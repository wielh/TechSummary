# SchemaSpy - 資料庫架構視覺化分析工具

[SchemaSpy](https://schemaspy.org/) 是一款基於 Java 的開源工具，能夠自動分析資料庫的 Schema 並生成互動式的 HTML 報表。它不僅能列出資料表與欄位，還能視覺化地呈現表與表之間的關聯（ER 圖）。

## 1. 核心功能

- **互動式 ER 圖**：利用 Graphviz 生成動態的關聯圖，點選表名可跳轉至詳細說明。
- **異常檢測 (Anomalies)**：自動識別資料庫中潛在的問題，例如：
  - **孤兒表 (Orphan Tables)**：沒有任何外鍵關聯的表。
  - **隱含的關聯**：根據欄位名稱（如 `user_id`）推測可能存在的關聯。
- **搜尋與導航**：提供搜尋功能，可以快速找到特定的欄位、表或約束條件。
- **支援多種資料庫**：透過 JDBC 驅動程式，支援 PostgreSQL, MySQL, Oracle, SQL Server, SQLite 等主流資料庫。

---

## 2. 安裝與環境要求

1. **Java Runtime Environment (JRE)**：SchemaSpy 本身是 Java 程式，需要安裝 JRE 8 或以上版本。
2. **JDBC 驅動程式**：對應您資料庫的 `.jar` 驅動檔（例如 PostgreSQL 的 `postgresql-42.x.x.jar`）。
3. **Graphviz** (選配)：若要生成視覺化圖表，系統必須安裝 Graphviz 工具。

---

## 3. 基本使用範例

通常透過命令列執行：

```bash
java -jar schemaspy.jar \
  -t pgsql \
  -db <DATABASE_NAME> \
  -host <HOST_URL> \
  -port <PORT> \
  -u <USER_NAME> \
  -p <PASSWORD> \
  -o ./output \
  -dp <PATH_TO_JDBC_DRIVER>
```

**常用參數說明：**

- `-t`: 指定資料庫類型（如 `pgsql`, `mysql`）。
- `-o`: 報表輸出的目標路徑。
- `-dp`: 指定 JDBC 驅動程式路徑。

---

## 4. 使用 Docker 執行 (推薦)

使用 Docker 執行 SchemaSpy 可以省去安裝 Java、JDBC 驅動程式與 Graphviz 的麻煩，是目前最推薦的使用方式。

### 單次執行 (Single Run)

假設您的資料庫正在運行，可以透過以下指令生成報表：

```bash
docker run -v "$PWD/output:/output" --net="host" schemaspy/schemaspy \
  -t pgsql \
  -db <DATABASE_NAME> \
  -host localhost \
  -port 5432 \
  -u <USER_NAME> \
  -p <PASSWORD> \
  -o /output
```

### 使用 Docker Compose

若要與資料庫服務一同管理，可以在 `docker-compose.yml` 中加入：

```yaml
version: '3'
services:
  schemaspy:
    image: schemaspy/schemaspy:latest
    volumes:
      - ./schemaspy/output:/output
    command: [
      "-t", "pgsql",
      "-db", "mydb",
      "-host", "db_container_name",
      "-port", "5432",
      "-u", "user",
      "-p", "password",
      "-o", "/output"
    ]
    depends_on:
      - db
```

**注意事項：**

- **路徑掛載**：務必使用 `-v` 將本地目錄掛載至容器內的 `/output`，否則生成的報表會留在容器中無法讀取。
- **網路連接**：若資料庫在宿主機運行，需使用 `--net="host"`；若在 Docker 網路中，則使用容器名稱。

---

## 5. 為什麼要使用 SchemaSpy？

- **自動化文件**：無需手動繪製 ER 圖，資料庫變更後重新執行即可更新文件。
- **快速上手新專案**：新加入的開發者可以透過生成的 HTML 報表，快速理解複雜的資料庫架構。
- **SEO 與導航**：報表內的表名與欄位皆有連結，方便在大規模資料庫中快速追蹤外鍵關係。
- **輕量且免費**：不需安裝龐大的商用軟體，即可獲得高品質的資料庫文檔。
