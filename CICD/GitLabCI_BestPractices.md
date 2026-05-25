# GitLab CI/CD 生產級最佳實踐與安全指南

本指南基於生產級 CI/CD 配置（參考 `golang.gitlab-ci.yml`），提煉出關於**授權安全、部署路徑防護、日誌防洩漏**以及**測試覆蓋率視覺化**的核心工程實踐。

---

## 1. 私有模組授權驗證 (`.netrc` 機制)

### 背景問題

在 Go（或其他語言）開發中，專案經常會依賴內部 GitLab 的私有模組（Private Modules）。預設情況下，CI/CD Runner 執行 `go mod download` 時會因為缺乏授權而失敗。

### 解決方案：使用 `.netrc`

我們可以在 Job 啟動時，動態將憑證寫入當前使用者的 `~/.netrc` 檔案中。Go 官方工具鏈（及 `curl`、`git` 等）會自動讀取此檔來進行授權。

#### YAML 配置範例：

```yaml
variables:
  REGISTRY_HOSTNAME: "gitlab.example.com"

# 透過 YAML Anchor 定義初始化腳本
.token_setup: &token_setup
  - echo "machine ${REGISTRY_HOSTNAME} login gitlab-ci-token password ${KENDA_CI_ACCESS_TOKEN}" > ~/.netrc
  - chmod 600 ~/.netrc # 嚴格限制作業系統權限

test_job:
  stage: test
  script:
    - *token_setup # 引用並執行
    - go mod download
    - go test ./...
  after_script:
    - rm -f ~/.netrc 2>/dev/null || true # 確保不論成功或失敗，都將憑證檔案抹除
```

> 💡 **安全防護要點**：
>
> 1. `~/.netrc` 權限必須設為 `600`，防止同主機的其他 Process 讀取。
> 2. **務必在 `after_script` 中加入刪除指令**，防止憑證殘留在 Runner 本地磁碟中。

---

## 2. 防範路徑穿越安全檢查 (Path Traversal Prevention)

### 背景問題

在自動化部署（如透過 SSH 將檔案派送到遠端伺服器）的腳本中，常會使用環境變數（如 `DEPLOY_FOLDER`、`DEPLOY_CONFIG_DIR`）來動態拼接伺服器上的目標路徑（例如 `/srv/go/service/${DEPLOY_FOLDER}`）。

如果這些變數被惡意修改，或因配置失誤包含了 `..`（父目錄），腳本在執行類似 `tar -xf` 或 `mkdir` 時，就可能**將檔案寫入到非預期的系統目錄**（例如 `/etc/cron.d` 或 `/root/.ssh`），從而導致嚴重的系統漏洞。

### 解決方案：路徑消毒 (Sanitization)

在部署執行前，使用 Shell 的 `case` 語法強制檢查所有路徑變數是否包含 `..`：

```bash
# 驗證部署路徑變數是否安全
.validate_deploy_server: &validate_deploy_server
  - |
    # 防止路徑穿越 (Path Traversal)
    for _PATH_VAR in "${DEPLOY_FOLDER}" "${DEPLOY_CONFIG_TARGET:-}" "${DEPLOY_CONFIG_DIR:-}"; do
      case "${_PATH_VAR}" in
        *..*)
          echo "Error: 安全防護觸發！路徑變數不可包含 '..'（${_PATH_VAR}）"
          exit 1
          ;;
      esac
    done
```

---

## 3. 防範 Log 敏感資訊洩漏 (Log Security)

### 背景問題

在 CI 流程中，我們經常需要在部署完成後進行健康檢查（例如執行 `docker compose ps` 檢查容器是否存活）。

然而，容器啟動指令（Command）或環境變數中，經常包含資料庫連線字串 (DSN)、API 金鑰等敏感資訊。如果直接將狀態指令的輸出傾印到 CI 日誌，這些機密就會暴露在所有擁有 GitLab 檢視權限的人面前。

### 解決方案：僅過濾輸出特定欄位

不要直接 `cat` 或輸出完整的 JSON/文字狀態。應使用 `grep -oE` 或 `jq` 僅提取不具敏感性的欄位（如服務名稱 `Name` 與運行狀態 `State`）：

```bash
# 取得容器狀態 JSON，並利用 grep 過濾敏感欄位
CONTAINER_STATUS=$(ssh root@"${DEPLOY_SERVER}" "docker compose ps --format json")

# 只顯示 Name 和 State，防範 Command 欄位中的敏感帳密洩漏到 CI 日誌中
echo "${CONTAINER_STATUS}" | grep -oE '"(Name|State)":"[^"]*"'

# 判定是否正常
if echo "${CONTAINER_STATUS}" | grep -qiE '"State":"(exited|restarting|dead)"'; then
  echo "Error: 偵測到容器狀態異常，部署中止"
  exit 1
fi
```

---

## 4. Go 測試覆蓋率與 Cobertura 視覺化整合

### 什麼是 Cobertura XML？

Cobertura 原本是 Java 的測試覆蓋率工具，但其產出的 XML 格式已成為軟體界描述覆蓋率的標準格式。

GitLab 內建支援解析 Cobertura XML 報告。只要在 CI 中提供此報告，GitLab 就能在 **Merge Request (MR) 程式碼差異 (Diff) 畫面中，以綠色（已覆蓋）與紅色（未覆蓋）直接呈現每一行程式碼的測試狀況**。這能極大提升 Code Review 的效率與測試防護。

### 在 Go 專案中的配置步驟

Go 語言內建的 `go test -coverprofile` 輸出格式是文字檔，我們需要使用開源工具 `gocover-cobertura` 將其轉換為 Cobertura 格式的 XML。

```yaml
test_job:
  stage: test
  image: golang:1.21
  script:
    # 1. 執行單元測試並輸出 Go 原生覆蓋率檔案
    - go test -coverprofile=coverage.txt -race ./...

    # 2. 安裝轉換工具並進行轉換
    - go install github.com/boumenot/gocover-cobertura@latest
    - gocover-cobertura < coverage.txt > coverage.xml

    # 3. 輸出總覆蓋率文字（讓 GitLab 正則表達式抓取）
    - go tool cover -func=coverage.txt

  # 正則表達式，用以在 GitLab 顯示總覆蓋率數值
  coverage: '/^total:[ \t]+\(statements\)[ \t]+\d+.\d+%$/'

  # 上傳報告產物，使 GitLab MR 頁面可以直接渲染綠色/紅色覆蓋率視覺化線條
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
```
