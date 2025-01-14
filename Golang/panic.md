# panic

## 一些原因

在 Go 程式中，panic 通常表示異常或不可恢復的錯誤，例如程式邏輯錯誤或不可預期的情況。以下是可能導致 panic 的常見情況：

+ 明確呼叫 panic

+ 除零操作

+ 陣列或切片越界

+ 空指標解引用

+ 類型斷言失敗

+ 使用 close 關閉已關閉的通道

+ 讀寫 nil 通道

+ runtime 錯誤: Go 程式執行時發生的低層錯誤，通常由 Go runtime 自動觸發。例如：栈溢出（stack overflow），內存分配失敗

+ 使用未初始化的 sync.Mutex 或 RWMutex

+ JSON 解析中的非法數據

+ sync 计数为负数。

+ 类型断言不匹配。

## 捕捉 panic 範例

```
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered from panic:", r)
    }
}()
```