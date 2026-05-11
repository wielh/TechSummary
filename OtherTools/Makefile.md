# Makefile 簡介

Makefile 是一款經典的自動化建置工具，透過定義一系列的規則（Rules）來指定如何編譯程式、跑測試或執行例行性的開發任務。在 Go 專案中，它常被用作「專案任務管理員」，將複雜的指令簡化。

---

## 1. 基本語法結構

一個標準的 Makefile 規則由以下三部分組成：

```makefile
target: prerequisites
    recipe
```

- **target (目標)**：通常是要產生的檔案名稱，或是任務名稱（如 `build`）。
- **prerequisites (前置條件)**：執行此規則前必須先完成的其他目標或存在的檔案。
- **recipe (指令)**：實際執行的 Shell 指令。**必須以一個 Tab 鍵縮進**。

---

## 2. Go 專案實戰範本

以下是一份典型的 Go 專案 Makefile：

```makefile
# 變數定義
BINARY_NAME=myapp
VERSION=1.0.0

.PHONY: all build test clean run gen

# 預設任務
all: build test

# 編譯
build:
    go build -o $(BINARY_NAME) -v

# 執行測試 (包含禁用內聯優化以支援 monkey patch)
test:
    go test ./... -v -count=1 -gcflags="all=-l"

# 執行程式碼生成
gen:
    go generate ./...

# 清理編譯產物
clean:
    go clean
    rm -f $(BINARY_NAME)

# 執行專案
run: build
    ./$(BINARY_NAME)
```

---

## 3. 關鍵技巧說明

### A. .PHONY (偽目標)

如果你的專案目錄下剛好有一個檔案叫 `build`，執行 `make build` 時 Make 會認為檔案已存在而跳過任務。
使用 `.PHONY` 可以告訴 Make：**這些是任務名稱，不是檔案名稱**。

```makefile
.PHONY: build test clean
```

### B. 變數與賦值

可以使用 `$(VARIABLE)` 來引用變數，方便統一管理版本號、路徑或編譯參數。

### C. 靜默執行 `@`

在指令前加上 `@`，執行時就不會印出指令本身，只會印出結果。

```makefile
help:
    @echo "Available commands:"
    @echo "  make build - Build the binary"
```

---

## 4. 為什麼要用 Makefile？

1. **標準化開發流程**：不論是新成員加入或 CI 環境，只需輸入 `make test` 就能跑測試。
2. **簡化長指令**：將帶有複雜 `gcflags` 或 `ldflags` 的指令縮減為一個單字。
3. **任務依賴**：例如 `run` 任務可以設定依賴於 `build`，確保執行前一定會先編譯。
4. **跨語言一致性**：無論專案是用 Go、Rust 還是 Python 寫的，進入點都可以統一為 `make`。
