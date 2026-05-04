# Go Build 編譯指南

針對執行檔體積、效能與部署相容性的編譯技巧。

---

## 1. 基礎編譯指令

```bash
go build -o app_name main.go
```

---

## 2. 進階編譯參數

### A. 靜態編譯與環境相容性

為了讓執行檔能在極簡的 Docker 映像檔（如 `scratch` 或 `alpine`）運行，需確保無外部依賴。

* **環境變數**：`CGO_ENABLED=0`
  * **作用**：完全禁用 CGO，使用 Go 內建的連結器產生純靜態執行檔。
* **參數**：`-ldflags="-extldflags=-static"`
  * **核心機制**：當必須使用 `CGO_ENABLED=1` 時（例如引用了 C 函式庫），此參數會指示「外部連結器」（如 `gcc`）將 C 語言相依項也靜態打包進去。
  * **解決痛點**：防止在不同 Linux 發行版間移動執行檔時出現 `GLIBC not found` 的相依性錯誤。

### B. 體積優化與瘦身 (`-ldflags="-s -w"`)

* **`-s` (Omit Symbol Table)**：移除符號表。這會導致無法使用 `nm` 或 `gdb` 查看程式內的函式與變數名稱。
* **`-w` (Omit DWARF)**：移除 DWARF 除錯資訊。這會導致無法使用 `dlv` (Delve) 等除錯器進行追蹤。
* **效果**：減少執行檔體積約 20%~30%，但會喪失對 binary 進行底層除錯的能力。
* **注意**：發生 Panic 時，基本的 Stack Trace 仍然存在，但資訊量會減少。

### C. 動態資訊注入

在編譯時將 CI 變數（如 Version, Commit SHA）注入程式碼：

```bash
go build -ldflags "-X main.version=${CI_COMMIT_TAG} -X main.buildTime=$(date +%Y-%m-%dT%H:%M:%S)"
```

### **停用內聯 (`-gcflags -l`)**

在需要精確調試或使用某些特定 Mock 框架（需取得函數位址）時使用，防止編譯器自動將小函數展開。

---

## 3. 跨平台建置 (Cross-Compilation)

直接在當前環境產生其他平台的執行檔：

* **Linux (64-bit)**: `GOOS=linux GOARCH=amd64 go build ...`
* **Windows (64-bit)**: `GOOS=windows GOARCH=amd64 go build ...`
* **macOS (Apple Silicon)**: `GOOS=darwin GOARCH=arm64 go build ...`

---
