# GitLab CI/CD 驗證方式：CI_JOB_TOKEN vs. SSH Key

在 GitLab CI 流程中，當 Job 需要存取其他資源（如其他專案的程式碼、Docker Registry 或遠端伺服器）時，通常會用到這兩種驗證方式。

---

## 1. CI_JOB_TOKEN

`CI_JOB_TOKEN` 是 GitLab CI/CD 在每個 Job 開始時自動產生的臨時權限憑證 (Token)。

### 特點

* **自動化**：不需額外設定，可直接透過環境變數 `$CI_JOB_TOKEN` 使用。
* **短暫性**：權限僅在該 Job 執行期間有效，Job 結束後立即失效。
* **範圍限制**：主要用於存取同一個 GitLab 實例內的資源（如 API, Packages, Container Registry）。

### 常見用途

* **存取同一 GitLab 下的其他專案**：
  
    ```yaml
    clone_other_project:
      script:
        - git clone https://gitlab-ci-token:${CI_JOB_TOKEN}@gitlab.example.com/other-group/other-project.git
    ```

* **登入 Container Registry**：

    ```yaml
    docker login -u gitlab-ci-token -p $CI_JOB_TOKEN $CI_REGISTRY
    ```

---

## 2. SSH Key

傳統的非對稱加密驗證方式，通常用於 CI 需要與外部伺服器或非同平台的 Git 服務通訊時。

### 特點

* **長期有效**：除非手動撤銷或更換金鑰，否則權限一直存在。
* **通用性**：適用於任何支援 SSH 的伺服器（如 Ubuntu, CentOS）或 Git 服務（如 GitHub）。
* **需手動設定**：需將私鑰存入 GitLab CI/CD Variables，並將公鑰存入目標端。

### 常見用途

* **佈署到遠端伺服器 (CD)**：
    ```yaml
    deploy:
      before_script:
        - eval $(ssh-agent -s)
        - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
      script:
        - ssh user@server "cd /app && git pull"
    ```

---

## 3. 詳細比較

| 特性           | CI_JOB_TOKEN               | SSH Key                     |
| :------------- | :------------------------- | :-------------------------- |
| **生命週期**   | **短暫 (Job 結束即失效)**  | **長期 (手動撤銷前有效)**   |
| **管理複雜度** | **低 (全自動)**            | **高 (手動產生與存放)**     |
| **安全性**     | **極高** (不怕 Token 洩漏) | **中** (私鑰洩漏需立即更換) |
| **適用範圍**   | **GitLab 內部生態系**      | **跨平台、跨伺服器**        |
| **常見協定**   | HTTPS (HTTPS API/Git)      | SSH (Git over SSH/Terminal) |

---

## 4. 決策建議

* **內部互通首選 `CI_JOB_TOKEN`**：如果要存取同一個 GitLab 內的依賴項或 Registry，不要用 SSH Key。
* **外部佈署必用 `SSH Key`**：如果要連線到公司的 Production Server 或外部雲端主機，SSH Key 是標準做法。
* **跨平台整合**：如果 CI 需要去 GitHub 抓東西，則需使用 SSH Key 或 GitHub 的 Personal Access Token。
