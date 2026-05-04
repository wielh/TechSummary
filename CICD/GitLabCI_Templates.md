# GitLab CI/CD Templates (模板與模組化)

當專案變多、CI 邏輯變複雜時，將所有內容寫在同一個 `.gitlab-ci.yml` 會變得難以維護。Template 的核心目的是**程式碼重用 (Reuse)**。

---

## 1. 核心關鍵字：`include`

`include` 允許你將外部的 YAML 檔案引入到當前的配置中。

### A. 引入本地檔案 (`local`)

引入同一個專案路徑下的檔案。

```yaml
include:
  - local: '/templates/golang-build.gitlab-ci.yml'
```

### B. 引入其他專案檔案 (`file`)

引入同一個 GitLab 實例中其他專案的檔案。
```yaml
include:
  - project: 'my-group/ci-templates'
    file: '/templates/docker-build.yml'
```

### C. 引入遠端檔案 (`remote`)

透過公開連結引入外部 YAML。

```yaml
include:
  - remote: 'https://example.com/ci-template.yml'
```

### D. 引入 GitLab 內建模板 (`template`)

GitLab 官方提供的常用模板（如 Auto DevOps, 特定語言的 Linter）。

```yaml
include:
  - template: Jobs/Dependency-Scanning.gitlab-ci.yml
```

---

## 2. 擴充與覆寫：`extends`

當你引入一個模板後，你可能需要針對特定 Job 進行微調。

### 基本用法

`extends` 可以讓你繼承一個「隱藏任務」（名稱以 `.` 開頭的任務），並複寫其中的屬性。

```yaml
# 模板定義 (template.yml)
.test-template:
  stage: test
  script:
    - go test ./...
  retry: 2

# 實際使用
unit-test:
  extends: .test-template
  script:
    - go test -v ./...  # 覆寫原本的 script，但保留 retry: 2
```

---

## 3. 繼承 vs. 合併 (Inheritance vs. Merging)

當使用 `include` 或 `extends` 時，GitLab 會進行「深層合併 (Deep Merge)」：

1. **不同屬性會累加**：例如模板定義了 `image`，你的 Job 定義了 `variables`，結果會兩者都有。
2. **相同屬性會覆寫**：你的 Job 定義的內容優先權高於模板。
3. **Array (陣列) 注意事項**：如 `script` 或 `tags` 是陣列，通常會被**完整替換**，而不是合併元素（除非使用特殊的語法）。

---

## 4. 最佳實踐：隱藏任務 (Hidden Jobs)

以點 `.` 開頭的 Job 名稱不會被 GitLab Runner 執行，它們專門用來作為模板。

```yaml
.deploy-logic:
  script:
    - echo "Deploying to $ENV"
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

deploy-prod:
  extends: .deploy-logic
  variables:
    ENV: "production"
```

---

## 5. 為什麼要用 Template？

1. **統一規範**：全公司可以共用同一個安全掃描或編譯腳本。
2. **降低重複**：不需要在每個專案都貼上幾百行的 CI 程式碼。
3. **集中維護**：當工具版本升級時，只需修改 Template 專案，所有使用該 Template 的專案都會自動生效。
