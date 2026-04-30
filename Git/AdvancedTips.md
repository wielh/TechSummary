# Git 進階技巧 (Advanced Tips)

掌握這些進階指令可以大幅提升解決問題的效率。

## 1. 暫存變更 (Stash)

當你工作到一半需要切換分支，但又不想 commit 時：

- **git stash**: 將當前變更暫存起來。
- **git stash list**: 查看暫存列表。
- **git stash pop**: 取出最後一次暫存並移除紀錄。
- **git stash apply**: 取出最後一次暫存但保留紀錄。

## 2. 挑選提交 (Cherry-pick)

只想把某個分支的「某一個」提交套用到當前分支：

```bash
git cherry-pick <commit_id>
```

## 3. 救命法寶 (Reflog)

如果你不小心刪除了分支或遺失了 commit，Git 其實都有記錄：

- **git reflog**: 查看本地端所有的操作紀錄（包含已被刪除的 commit）。
- **git reset --hard <commit_id>**: 回到 reflog 中找到的那個狀態。

## 4. 互動式重基 (Interactive Rebase)

用來整理、合併、修改之前的 Commit 歷史：

```bash
git rebase -i HEAD~3
```

常見關鍵字：
- `pick`: 保留該 commit。
- `reword`: 修改 commit 訊息。
- `squash`: 將該 commit 合併到前一個。
- `drop`: 刪除該 commit。

## 5. 檔案搜尋與追蹤

- **git blame <file>**: 查看每一行程式碼是誰在什麼時候寫的。
- **git clean -fd**: 刪除工作區中未被追蹤的檔案與目錄。
