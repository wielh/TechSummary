# VS Code (Visual Studio Code) 簡介

Visual Studio Code (簡稱 VS Code) 是由微軟開發的一款開源、跨平台且輕量級的原始碼編輯器。它結合了編輯器的簡約與 IDE 的強大功能，是目前全球開發者最受歡迎的工具之一。

---

## 1. 核心優勢

- **高效能與輕量**：相較於傳統 IDE，啟動速度極快，記憶體消耗相對較低。
- **強大的生態系**：擁有數萬個擴充外掛 (Extensions)，能支援幾乎所有程式語言。
- **內建 Git 整合**：直接在編輯器內即可進行 Commit, Push, Pull 與解決衝突。
- **智慧感應 (IntelliSense)**：提供語法高亮、自動補全、參數提示以及跳轉至定義。

---

## 2. Go 開發建議配置

針對 Go 語言開發，建議安裝以下外掛與進行配置：

### 必備外掛

- **Go (by Google)**：官方提供的外掛，支援測試、除錯、語法檢查與導航。

### 建議設定

在 `settings.json` 中加入以下設定以提升體驗：

```json
{
    "go.formatTool": "goimports",
    "go.useLanguageServer": true,
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.organizeImports": true
    }
}
```

---

## 3. 常用快捷鍵 (Windows/Linux)

| 快捷鍵                 | 作用                           |
| :--------------------- | :----------------------------- |
| **`Ctrl + P`**         | 快速搜尋並開啟檔案             |
| **`Ctrl + Shift + P`** | 開啟指令面板 (Command Palette) |
| **`Ctrl + ` `**        | 切換終端機面板                 |
| **`Alt + Shift + F`**  | 格式化程式碼                   |
| **`F12`**              | 前往定義 (Go to Definition)    |
| **`Ctrl + Shift + F`** | 全域搜尋文字                   |

---

## 4. 推薦的進階功能

- **Remote Development**：透過 SSH, Container 或 WSL 直接在遠端環境開發，體驗與本地完全一致。
- **GitHub Copilot**：AI 輔助寫程式，能根據上下文自動生成代碼片段。
- **Settings Sync**：登入 GitHub 帳號即可跨裝置同步你的所有設定與外掛。
