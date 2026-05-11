# VS Code 專案配置詳解

本指南旨在說明如何透過 `.vscode` 資料夾內的設定檔，實現團隊開發環境的標準化與自動化。

---

## 1. .vscode/settings.json

此檔案用於定義專案專屬的編輯器行為。它會覆蓋使用者的全局設定，確保團隊成員在處理同一份程式碼時，體驗與產出是一致的。

### A. 編輯器基礎行為

| 設定項                           | 作用描述                           | 推薦理由                                                           |
| :------------------------------- | :--------------------------------- | :----------------------------------------------------------------- |
| **`"editor.formatOnSave"`**      | 存檔時自動觸發排版工具。           | 確保提交到 Git 的程式碼風格永遠符合團隊規範。                      |
| **`"editor.tabSize"`**           | 定義縮進的空格數（如 2 或 4）。    | 避免不同開發者混用不同縮進導致的排版混亂。                         |
| **`"files.insertFinalNewline"`** | 存檔時在檔案末尾自動插入一個空行。 | 符合 Unix 標準，避免 Git Diff 出現 `\ No newline at end of file`。 |

### B. 自動化操作 (Code Actions)

```json
"editor.codeActionsOnSave": {
    "source.organizeImports": true
}
```

- **作用**：存檔時自動執行程式碼清理。
- **優點**：會自動刪除未使用的 Import 並進行排序，保持程式碼簡潔，避免編譯錯誤。

### C. 檔案過濾與雜訊消除

```json
"files.exclude": {
    "**/.git": true,
    "**/node_modules": true,
    "**/dist": true,
    "**/*.exe": true
}
```

- **作用**：在左側檔案瀏覽器 (Explorer) 中隱藏特定目錄，且全域搜尋時會自動跳過這些路徑。
- **推薦理由**：減少搜尋時的無效結果，並保護 `.git` 等核心敏感目錄不被誤觸。

### D. Go 語言專屬優化

- **`"go.useLanguageServer": true`**：啟用官方的 `gopls`。能大幅提升跳轉定義、類型提示與自動補全的速度。
- **`"go.testFlags": ["-v", "-count=1"]`**：
  - `-v`：強制顯示詳細測試日誌。
  - `-count=1`：停用測試快取，確保每次點選 `run test` 都是最新的執行結果。
- **`"go.lintFlags": ["--fast"]`**：設定 Linter 指令，能在背景快速檢查潛在的邏輯錯誤。

---

## 2. .vscode/extensions.json

此檔案定義了該專案建議使用的工具清單。

### 運作機制

當新成員第一次開啟此專案時，VS Code 會在視窗右下角跳出提示：「此工作區有推薦的擴充外掛」。點選安裝後，所有必備工具即可一鍵到位。

### 配置範例與說明

```json
{
    "recommendations": [
        "golang.go",              // 必要：Go 語言核心支援
        "eamodio.gitlens",        // 增強：Git 歷史紀錄與責任追溯
        "usernamehw.errorlens",   // 輔助：直接在代碼行尾顯示報錯訊息
        "ms-azuretools.vscode-docker", // 工具：支援 Dockerfile 語法與容器管理
        "streetsidesoftware.code-spell-checker" // 品質：防止變數命名拼錯
    ]
}
```

---

## 3. 實作建議與最佳實踐

1. **版本控制**：強烈建議將 `.vscode/` 資料夾加入 Git（除非包含個人敏感的路徑）。這能讓新成員實現「Clone 即開發」的體驗。
2. **去個人化**：在 `settings.json` 中，應盡量避免寫入帶有個人電腦絕對路徑的設定（如 `C:/Users/Bo/...`），而應使用 `${workspaceFolder}` 等變數。
3. **拼字檢查字典**：若專案有特有的業務名詞，可在 `settings.json` 中加入 `"cSpell.words": ["placeholder"]`，確保這些詞不會被標示為錯誤。
