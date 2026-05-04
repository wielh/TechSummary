# YAML Anchors (&) 與 Aliases (*) 語法說明

這是一種 YAML 標準語法，用於在同一個檔案內**複製與重用**區塊，減少重複勞動。在 GitLab CI 中，它常被用來定義通用的腳本或設定。

---

## 1. 基本語法

* **`&` (Anchor)**：定義一個「錨點」，即標記一段你想要重複使用的內容。
* **`*` (Alias)**：使用「別名」來引用之前定義的錨點。
* **`<<:` (Merge Key)**：將錨點的內容「合併」到當前位置。

### 簡單範例

```yaml
.common_config: &common_settings
  image: alpine:latest
  retry: 2

job1:
  <<: *common_settings  # 複製 common_settings 的內容到這裡
  script:
    - echo "Doing job 1"

job2:
  <<: *common_settings
  script:
    - echo "Doing job 2"
```

---

## 2. 進階用法：重複使用腳本片段

你也可以針對單一屬性（如 `script`）定義錨點：

```yaml
.setup_scripts: &init_env
  - apk add --no-cache curl
  - export VERSION=$(cat version.txt)

test_job:
  image: alpine:latest
  script:
    - *init_env  # 直接引用腳本列表
    - echo "Running tests for version $VERSION"
```

---

## 3. 與 `extends` 的差異

雖然這兩者看起來很像，但它們在 GitLab CI 中的運作機制不同：

| 特性           | YAML Anchors (`&` / `*`)             | GitLab `extends`                        |
| :------------- | :----------------------------------- | :-------------------------------------- |
| **處理層級**   | 在 YAML 解析階段處理（最底層）       | 在 GitLab 邏輯處理階段處理              |
| **跨檔案支援** | **不支援**（只能在同一個檔案內使用） | **支援**（可配合 `include` 跨檔案使用） |
| **合併邏輯**   | 簡單的複製貼上，複寫邏輯較生硬       | 智慧型的深層合併 (Deep Merge)           |

---

## 4. 為什麼你還是會看到它？

儘管 GitLab 官方推薦優先使用 `extends`（因為支援跨檔案且邏輯更清晰），但在以下場景 Anchor 依然無可替代：

1. **重用 script 片段**：`extends` 只能繼承整個 Job 的屬性，而 Anchor 可以只重用 `script` 陣列中的某幾行。
2. **重用特定的變數群組**。
3. **在不支援 `extends` 的舊版 GitLab 環境中**。

---
