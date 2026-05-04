# GitLab CI/CD 配置文件結構 (.gitlab-ci.yml)

`.gitlab-ci.yml` 是 GitLab CI/CD 的核心配置文件，採用 YAML 格式，定義了自動化流水線 (Pipeline) 的行為。

---

## 1. 基本組成元素

一個基礎的 CI 配置文件通常包含以下部分：

### A. Stages (階段)

定義流水線的執行順序。同一個 Stage 裡的 Job 會並行執行，前一個 Stage 成功後才會進入下一個 Stage。

```yaml
stages:
  - test
  - build
  - deploy
```

### B. Image (映像檔)

指定執行 Job 時所使用的 Docker 映像檔。

```yaml
image: node:latest  # 全域設定，所有 Job 預設使用此映像檔
```

### C. Jobs (任務)

自定義的任務名稱，包含執行的具體指令。

```yaml
job_name:
  stage: build      -- 指定屬於哪個階段
  script:
    - echo "Building..."
```

---

## 2. 關鍵關鍵字說明

| 關鍵字              | 說明                                                                     |
| :------------------ | :----------------------------------------------------------------------- |
| **`script`**        | **唯一必填項目**。定義要執行的 Shell 指令。                              |
| **`before_script`** | 在 `script` 執行前運行的指令（常用於環境初始化）。                       |
| **`after_script`**  | 在 `script` 執行後運行的指令（不論成功或失敗都會執行）。                 |
| **`variables`**     | 定義環境變數。                                                           |
| **`rules`**         | 定義 Job 執行的條件（例如：只有在 merge request 時執行）。               |
| **`artifacts`**     | 指定 Job 結束後要保留的檔案或目錄（可傳遞給下一個 Stage）。              |
| **`cache`**         | 用於在多個 Job 或 Pipeline 之間共享依賴項（如 node_modules），提升速度。 |

---

## 3. 完整範例

```yaml
# 全域變數
variables:
  APP_NAME: "my-app"

stages:
  - build
  - test

# 編譯任務
build-job:
  stage: build
  image: golang:1.21
  script:
    - go build -o $APP_NAME
  artifacts:
    paths:
      - $APP_NAME  # 將編譯好的二進位檔傳給下一個 Stage

# 測試任務
unit-test-job:
  stage: test
  image: golang:1.21
  script:
    - go test ./...
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event" # 僅在 MR 時執行
```

---

## 4. 常見觀念

1. **Pipeline**: 整個 `.gitlab-ci.yml` 執行的一次完整流程。
2. **Runner**: 負責執行 Job 的機器（Agent）。
3. **依賴關係**: 預設情況下，下一個 Stage 會自動下載上一個 Stage 產出的 `artifacts`。
4. **YAML 語法**: 注意縮進 (Indentation)，YAML 對空格非常敏感。
