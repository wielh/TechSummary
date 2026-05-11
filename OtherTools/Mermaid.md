# Mermaid 簡介

Mermaid 是一款基於 JavaScript 的圖表工具，它使用類 Markdown 的文字語法來定義圖表，並自動渲染成 SVG 或 Canvas 格式。

## 1. 為什麼要使用 Mermaid？

- **文字即圖表**：不需要使用複雜的繪圖軟體，只需寫簡單的文字。
- **版本控制友好**：圖表定義是純文字，可以輕鬆放入 Git 進行版本追蹤與 Diff 對比。
- **Markdown 整合**：大多數現代 Markdown 編輯器（如 VS Code, GitHub, GitLab, Obsidian）都原生支援 Mermaid 渲染。
- **維護成本低**：修改圖表只需修改幾行文字，不需要重新匯出圖片。

---

## 2. 常用圖表範例

### A. 流程圖 (Flowcharts)

用於展示邏輯判斷或操作流程。

```mermaid
graph TD
    A[開始] --> B{是否登入?}
    B -- 是 --> C[進入首頁]
    B -- 否 --> D[跳轉登入頁]
    C --> E[結束]
```

### B. 序列圖 (Sequence Diagrams)

常用於描述物件或服務之間的互動時序。

```mermaid
sequenceDiagram
    participant User
    participant App
    participant DB
    
    User->>App: 輸入帳密
    App->>DB: 驗證用戶
    DB-->>App: 回傳結果 (Success)
    App-->>User: 登入成功
```

### C. 甘特圖 (Gantt Charts)

用於專案時程管理。

```mermaid
gantt
    title 專案開發進度
    dateFormat  YYYY-MM-DD
    section 設計階段
    需求分析           :a1, 2023-10-01, 5d
    UI 設計           :after a1, 7d
    section 開發階段
    後端開發           :2023-10-10, 10d
    前端開發           :2023-10-12, 8d
```

### D. 類別圖 (Class Diagrams)

用於描述程式架構中的類別關係。

```mermaid
classDiagram
    Animal <|-- Duck
    Animal <|-- Fish
    Animal : +int age
    Animal : +String gender
    Animal: +isMammal()
    class Duck{
        +String beakColor
        +swim()
        +quack()
    }
```

---

## 3. 使用建議

1. **保持簡潔**：圖表過於複雜時會難以閱讀，建議將大圖拆分成多個子圖。
2. **語法練習**：可以利用 [Mermaid Live Editor](https://mermaid.live/) 進行即時編輯與預覽。
3. **搭配註釋**：在 Mermaid 代碼區塊上方加上簡短的說明文字，幫助他人理解圖表重點。
