# Git 基本指令 (Basic Commands)

Git 是一個分散式版本控制系統，以下是開發中常用的基本指令。

## 1. 初始化與複製

- **git init**: 在當前目錄初始化一個新的 Git 倉庫。
- **git clone <url>**: 複製遠端倉庫到本地。

## 2. 檢查狀態

- **git status**: 查看工作區與暫存區的狀態。
- **git diff**: 查看尚未暫存的變動內容。

## 3. 提交變更

- **git add <file>**: 將檔案加入暫存區 (Staging Area)。
- **git add .**: 將所有變動檔案加入暫存區。
- **git commit -m "message"**: 提交暫存區的變更並記錄訊息。
- **git commit --amend**: 修改最後一次提交的訊息或內容。

## 4. 同步遠端

- **git fetch**: 取得遠端倉庫的新資訊（不合併）。
- **git pull**: 取得並合併遠端分支的變更。
- **git push**: 將本地提交推送到遠端倉庫。

## 5. 查看歷史

- **git log**: 查看提交歷史記錄。
- **git log --oneline --graph --all**: 以圖形化且簡潔的方式查看所有分支歷史。
- **git show <commit_id>**: 查看特定提交的詳細變動內容。
