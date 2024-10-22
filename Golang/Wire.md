# google/wire

## 作用: 

在系統日漸壯大的情況下，手動撰寫依賴注入(DI)的成本會越來越高，需要建立的實例越來越多。wire 讓我們更方便管理依賴。

## Example

+ 手動注入

```
func NewMessage() Message {
    return Message("Hi there!")
}
func NewGreeter(m Message) Greeter {
    return Greeter{Message: m}
}
func NewEvent(g Greeter) Event {
    return Event{Greeter: g}
}

func main() {
    message := NewMessage()
    greeter := NewGreeter(message)
    event := NewEvent(greeter)

    event.Start()
}
```

+ 使用 wire

```
// wire.go

func InitializeEvent() Event {
    wire.Build(NewEvent, NewGreeter, NewMessage)
    return Event{}
}
```

```
// 在目錄下執行指令 wire 或是 wire gen，接著會生成檔案 wire_gen.go
func InitializeEvent() Event {
    message := NewMessage()
    greeter := NewGreeter(message)
    event := NewEvent(greeter)
    return event
}
```

## 補充

+ 不能夠有多個建構子回傳同一種類型

+ 建構子回傳類型的部分被 wire 稱作 provider，生成的初始化建構函示則稱為 injector

+ 在執行完 wire 生成 code 之後會發現 InitializeEvent 亮紅字，而且沒辦法執行程式，因為 InitializeEvent 會由於產生出來的程式碼而重複宣告，解法是在 wire.go 上加入一個註解

## reference

+ https://go-kratos.dev/en/blog/