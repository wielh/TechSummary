# GitLab CI/CD 中的 Cache (快取) vs. Artifacts (產物)

在編寫 `.gitlab-ci.yml` 流水線時，`cache` 與 `artifacts` 是最常用於管理檔案傳遞與加速編譯的兩個關鍵字。理解它們的核心差異，能有效提升 Pipeline 的執行速度並避免檔案丟失。

---

## 1. 核心觀念對比

這兩者的核心目的截然不同：**Cache 是為了「加速」，Artifacts 是為了「傳遞與交付」**。

| 特性 | Cache (快取) | Artifacts (產物) |
| :--- | :--- | :--- |
| **主要目的** | **加速後續 Pipeline 的編譯速度**。 | **傳遞階段成果**，或下載最終交付物。 |
| **適用內容** | 專案依賴套件（如 `node_modules`、Go module 緩存、編譯中間檔）。 | 編譯出來的二進位檔、打包後的前端 dist、測試與覆蓋率報告。 |
| **生命週期** | 跨 Pipeline、跨分支持久存在（通常由 Runner 或 S3 快取機制管理）。 | **只在同一次 Pipeline 內跨 Stage 傳遞**，過期後會被 GitLab 自動清理。 |
| **可靠性** | **不保證一定存在**（快取可能會失效或被主動清理，Job 必須寫得「即使沒有快取也能重新下載」）。 | **強保證存在**（只要前一個 Stage 成功，下一個 Stage 必定能下載到該產物）。 |
| **儲存位置** | 分散式 Object Storage (如 S3, MinIO) 或 Runner 本地磁碟。 | **GitLab Server 伺服器端**（可直接在 GitLab 網頁 UI 上下載）。 |

---

## 2. 常見配置誤區

1. **❌ 錯誤：使用 `cache` 傳遞編譯後的程式碼給下一個 Stage 部署**
   * *原因*：快取是不穩定的。若下一個 Stage 分配到不同的 Runner，且快取同步失敗，部署 Job 就會找不到編譯好的二進位檔。**傳遞編譯結果請務必使用 `artifacts`**。
2. **❌ 錯誤：將依賴庫（如 `node_modules`）放入 `artifacts` 傳遞**
   * *原因*：`artifacts` 會被打包上傳到 GitLab Server，檔案過大會極大增加網路傳輸時間並塞爆 GitLab 伺服器的磁碟空間。**下載的依賴庫請務必使用 `cache`**。

---

## 3. Go 語言實戰配置範例 (`go mod download`)

在 Go 專案中，我們希望：
1. **快取 (Cache)**：將 Go 的依賴庫（`go mod`）與編譯快取（`go build` 快取）保存下來，避免每次 Job 都重新去網路上抓套件與重新編譯所有代碼。
2. **產物 (Artifacts)**：將編譯出來的二進位可執行檔傳遞給後續的 `deploy` 階段進行部署。

### 核心設定原理
* Go 預設的依賴下載路徑為 `$GOPATH/pkg/mod`，編譯快取路徑為 `$GOCACHE`。
* 為了讓 GitLab Runner 能成功將其快取，我們必須將這些路徑**導向到專案根目錄內**（例如變數設定為 `$CI_PROJECT_DIR/...`），因為 GitLab CI 的快取路徑限制只能在專案目錄內。

### `.gitlab-ci.yml` 完整配置範例

```yaml
image: golang:1.21

# 定義全域環境變數，將 Go 快取路徑導向至專案目錄內
variables:
  GOPATH: "$CI_PROJECT_DIR/.go"
  GOCACHE: "$CI_PROJECT_DIR/.gocache"

# 定義全域快取規則 (Key 通常使用分支或 go.sum 的 hash)
cache:
  key:
    files:
      - go.sum
  paths:
    - .go/pkg/mod/     # 快取 go mod 下載的第三方套件
    - .gocache/        # 快取 go build 的中間編譯產物

stages:
  - install
  - build
  - deploy

# 1. 下載依賴階段 (利用 cache 加速)
download-dependencies:
  stage: install
  script:
    - go mod download
  # 此 Job 結束後，GitLab Runner 會自動將 .go/pkg/mod 壓縮並上傳為快取

# 2. 編譯階段 (讀取並寫入 cache，輸出 artifacts)
build-binary:
  stage: build
  script:
    - go build -o my-app main.go
  # 將編譯產出物傳遞給 deploy 階段
  artifacts:
    name: "app-build-${CI_COMMIT_REF_SLUG}"
    expire_in: 1 week  # 產物保留一週，過期自動清理
    paths:
      - my-app         # 只有這個二進位檔會被上傳到 GitLab Server

# 3. 部署階段 (無需安裝 Go，只下載 artifacts 進行部署)
deploy-to-prod:
  stage: deploy
  image: alpine:latest  # 可以換成極小的部署映像檔
  dependencies:
    - build-binary      # 明確指定只下載 build-binary 任務的 artifacts
  script:
    - chmod +x ./my-app
    - ./my-app --version
    - echo "Deploying my-app to production server..."
```
