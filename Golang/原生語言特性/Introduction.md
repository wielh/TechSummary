# golang 語言特性

## 特性

+ 静态类型和强类型: Go 是静态类型的语言，这意味着在编译时进行类型检查。强类型则意味着不会进行隐式类型转换，从而避免了许多潜在的错误。

+ 内置并发支持: Go 的最大特点之一是内置了对并发的支持。使用 goroutines 和 channels，可以轻松实现并发编程。

+ 内嵌并发安全的数据结构: Go 标准库提供了一些内嵌并发安全的数据结构，如 sync.Map 和 sync.Pool，帮助开发者在多线程环境中更安全地操作数据。

+ 接口: Go 语言通过接口实现多态性。接口定义了一组方法，任何实现这些方法的类型都隐式地实现了该接口。(Duck Type)

+ 没有继承，使用组合: Go 不支持类的继承，而是提倡通过组合来实现代码重用。通过将一个类型嵌入到另一个类型中，可以复用其方法。

+ golang 並非使用 public, private, ...表示封裝級別，而是用方法/變數首字母為大小寫判斷，變數命名法為駝峰法。

+ 如果數據庫某欄位為 name，但因為 Golang 語言特性，首字母必須為大寫，那麼struct應該這樣定義:

    (json, mongodb的bson同理)

```
type message struct {
	ID       string     `pg:"id,pk"`
	RoomID   string     `pg:"room_id"`
	UserID   string
	Username string
	Time     time.Time
	Deleted  bool
	Content  string
}
```

    有些 package 會自動轉換欄位名(room_id <=> RoomID)，所以要先自己測試一下是否能找到欄位

## 關鍵字

+ defer: 用于推迟执行功能，直到周围的功能执行为止。

+ select: goroutine 在同步通信操作期间等待。

+ interface: 用于指定方法集。方法集是一种类型的方法列表。

+ goto: 可无条件跳转至带标签的语句。

+ fallthrough: 在 switch 语句中的情况下使用此关键字。当我们使用此关键字时，将输入下一种情况。

+ iota: 是一個特殊的常量生成器，用於簡化一組相關常量的定義。它從零開始，並且在每一個新的常量定義行中遞增。

    ```
    const (
        A = iota        // A == 0
        B               // B == 1
        C               // C == 2
    )
    ```

    ```
    const (
        A = 3*iota+2        // A == 2
        B                   // B == 5
        C                   // C == 8
        D = "hello"         // D == "hello"
        E                   // E == "hello" (與上行相同)
        F = iota            // F == 5 (恢復 iota 的遞增，並根據行數遞增)
        G                   // G == 6
    )
    ```



