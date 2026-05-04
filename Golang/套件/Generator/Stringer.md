# Stringer - 自動生成 Enum 的字串方法

`stringer` 是 Go 官方提供的一個代碼生成工具，專門用來為整數類型的列舉 (Enum) 自動生成 `String()` 方法。

## 1. 安裝

```bash
go install golang.org/x/tools/cmd/stringer@latest
```

---

## 2. 使用範例

假設你有一個代表訂單狀態的類型：

```go
package constants

//go:generate stringer -type=OrderStatus

type OrderStatus int

const (
    Pending OrderStatus = iota
    Processing
    Completed
    Cancelled
)
```

執行指令：

```bash
go generate ./...
```

---

## 3. 生成結果

`stringer` 會在同目錄下生成一個 `orderstatus_string.go` 檔案，內容大致如下：

```go
func (i OrderStatus) String() string {
    if i < 0 || i >= OrderStatus(len(_OrderStatus_index)-1) {
        return "OrderStatus(" + strconv.FormatInt(int64(i), 10) + ")"
    }
    return _OrderStatus_name[_OrderStatus_index[i]:_OrderStatus_index[i+1]]
}
```

---

## 4. 進階技巧：自定義字串

如果你希望生成的字串是小寫或是特定的格式，可以使用 `-linecomment` 參數並配合註解：

```go
//go:generate stringer -type=OrderStatus -linecomment

type OrderStatus int

const (
    Pending    OrderStatus = iota // 待處理
    Processing               // 處理中
    Completed                // 已完成
    Cancelled                // 已取消
)
```

這樣 `status.String()` 就會回傳註解中的中文內容。

---

## 5. 優點

1. **避免維護 switch-case**：不需要手動撰寫龐大的 `switch i { case Pending: return "Pending" ... }` 邏輯。
2. **類型安全**：若新增了列舉值，只需重新執行生成即可更新，不易遺漏。
3. **高效能**：產出的代碼使用預先計算的索引查找，效能極佳。
