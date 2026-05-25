# panic

## 一些原因

在 Go 程式中，panic 通常表示異常或不可恢復的錯誤，例如程式邏輯錯誤或不可預期的情況。以下是可能導致 panic 的常見情況：

+ 明確呼叫 panic

+ runtime 錯誤: Go 程式執行時發生的低層錯誤，通常由 Go runtime 自動觸發。例如：栈溢出（stack overflow），內存分配失敗

+ 除零操作

+ 陣列或切片越界

+ 空指標解引用

+ 類型斷言失敗

+ 使用 close 關閉已關閉的通道

+ 讀寫 nil 通道

+ 使用未初始化的 sync.Mutex 或 RWMutex

+ JSON 解析中的非法數據

+ sync 计数为负数。

+ 类型断言不匹配。

## 捕捉 panic 範例

```go
defer func() {
    if r := recover(); r != nil {
        fmt.Println("Recovered from panic:", r)
    }
}()
```

## 跨 Goroutine 捕獲失效

`recover` 只能捕獲**當前 Goroutine** 發生的 panic，無法跨 Goroutine 捕獲。如果在 `main` goroutine 中註冊了 `recover`，而子 goroutine 發生 panic，程式依然會直接崩潰。

### ❌ 錯誤範例 (會導致程式崩潰)

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // 這裡的 recover 無法捕獲子 goroutine 的 panic
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in main:", r)
        }
    }()

    go func() {
        panic("panic in goroutine") // 程式依然會崩潰！
    }()

    time.Sleep(1 * time.Second)
}
```

###  正確做法

必須在**每一個子 Goroutine 內部**獨立註冊 `recover`：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    go func() {
        // 在子 goroutine 內部進行捕獲
        defer func() {
            if r := recover(); r != nil {
                fmt.Println("Recovered in goroutine:", r)
            }
        }()
        
        panic("panic in goroutine") // 被成功捕獲，程式不會崩潰
    }()

    time.Sleep(1 * time.Second)
}
```