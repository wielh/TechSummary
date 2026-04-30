# Git 分支管理 (Branch Management)

分支是 Git 最強大的功能之一，允許開發者在不影響主線的情況下進行功能開發或修復。

## 1. 分支操作

- **git branch**: 列出本地所有分支。
- **git branch <name>**: 建立新分支。
- **git checkout <name>**: 切換到指定分支。
- **git checkout -b <name>**: 建立並直接切換到新分支。
- **git branch -d <name>**: 刪除已合併的分支。
- **git branch -D <name>**: 強制刪除分支。

## 2. 合併與重基 (Merge vs Rebase)

> 關於兩者的詳細對比與實戰場景，請參考：[Merge vs. Rebase 詳細比較](file:///e:/code/TechSummary/Git/MergeVsRebase.md)

### Merge (合併)

將指定分支的歷史合併到當前分支，會產生一個新的 "Merge commit"。

```bash
git merge <feature-branch>
```

- **優點**: 完整保留開發歷史與順序。
- **缺點**: 當分支多時，歷史線條會變得混亂。

### Rebase (重基)

將當前分支的基準點移動到目標分支的最新提交上，使歷史呈現線性。

```bash
git rebase <main-branch>
```

- **優點**: 歷史線條非常乾淨、線性。
- **缺點**: 會改變 Commit ID，**嚴禁在多人共用的公共分支上使用 Rebase**。

## 3. 衝突解決 (Conflict Resolution)

當兩個分支修改了同一行程式碼時，合併會發生衝突。

1. 打開衝突檔案，尋找 `<<<<<<<`, `=======`, `>>>>>>>` 標記。
2. 決定要保留的內容，刪除標記。
3. `git add <file>` 標示為已解決。
4. `git commit` 完成合併。
