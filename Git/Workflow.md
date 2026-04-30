# Git 工作流 (Workflow)

良好的工作流規範能確保團隊協作順暢，減少衝突。

## 1. Git Flow

最經典的工作流，區分多個長期分支：

- **master (main)**: 存放隨時可發佈到生產環境的穩定程式碼。
- **develop**: 開發主線，存放預計併入下一次發佈的程式碼。
- **feature/**: 功能開發分支，從 develop 分出，完成後併回 develop。
- **release/**: 發佈準備分支，從 develop 分出，進行測試與 Bug fix，最後併回 master 與 develop。
- **hotfix/**: 緊急修復分支，直接從 master 分出，修復後併回 master 與 develop。

## 2. GitHub Flow

較簡化、適合持續部署 (CD) 的模式：

1. 從 `main` 分出新分支。
2. 進行開發並 Commit。
3. 發起 **Pull Request (PR)**。
4. 進行 Code Review 與討論。
5. 通過測試後合併回 `main` 並立即部署。

## 3. 提交訊息規範 (Conventional Commits)

建議遵循以下格式：`<type>(<scope>): <subject>`

- **feat**: 新功能。
- **fix**: 修補 Bug。
- **docs**: 文件更動。
- **style**: 不影響程式邏輯的格式更動（空白、縮排等）。
- **refactor**: 重構程式碼（非新功能、非 Bug 修復）。
- **perf**: 效能提升。
- **test**: 增加或修改測試。
- **chore**: 建置程序或輔助工具的變動。
