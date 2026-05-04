# Go 代碼檢查與安全指南 (Vet, Lint, Security)

在 CI 流程中確保代碼品質與安全性的必備檢查。

---

## 1. 基礎檢查 (go vet)

Go 內建的靜態分析工具，檢查常見錯誤（如 Printf 格式不符、無法觸及的代碼）。

```bash
go vet ./...
```

### 常見錯誤案例

* **Printf 格式錯誤**：

    ```go
    fmt.Printf("Hello %d", "world") // 應為 %s，vet 會抓出類型不符
    ```

* **無法觸及的程式碼 (Unreachable Code)**：

    ```go
    return
    fmt.Println("done") // vet 會報錯：unreachable code
    ```

* **閉包內錯誤使用迴圈變數**：

    ```go
    for _, v := range list {
        go func() { fmt.Println(v) }() // vet 會警告 loop variable v captured by func literal
    }
    ```

---

## 2. 靜態品質檢查 (golangci-lint)

目前社群最主流的 Lint 工具，整合了數十種檢查器。

* **執行指令**：
  
    ```bash
    golangci-lint run --timeout=5m
    ```

* **包含 CGO 專案**：如果專案有 C 語言調用，需加上標籤：

    ```bash
    golangci-lint run --build-tags cgo
    ```

### 常見錯誤案例

* **未處理的錯誤 (errcheck)**：

    ```go
    f, _ := os.Open("file.txt") // 忽略錯誤是不好的實踐
    defer f.Close()             // 甚至直接呼叫不接回傳值也會被抓
    ```

* **未使用到的變數或函式 (unused)**：

    ```go
    func unusedFunc() {} // 定義了卻沒人叫，會被要求刪除
    ```

* **複雜度過高 (gocyclo)**：

    如果一個函式內有太多的 `if-else` 或 `switch`，會被判定為維護困難。
* **空分支 (staticcheck)**：

    ```go
    if err != nil {
        // 什麼都不做
    }
    ```

---

## 3. 安全性檢查 (govulncheck)

Google 官方提供的工具，掃描你的依賴庫 (Dependencies) 中是否有已知的安全性漏洞。

* **安裝與執行**：

    ```bash
    go install golang.org/x/vuln/cmd/govulncheck@latest
    govulncheck ./...
    ```

---
