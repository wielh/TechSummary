# Channel 使用注意事項

Channel 是 Go 語言併發模型的關鍵，遵循「不要透過共享記憶體來溝通，而要透過溝通來共享記憶體」的原則。

---

## 1. Channel 狀態行為總結 (重要)

這是開發中最容易出錯的地方，請務必記住以下行為：

| 操作 | 未初始化 (nil) | 開啟中 (Open) | 已關閉 (Closed) |
| :--- | :--- | :--- | :--- |
| **讀取 `<-ch`** | **永久阻塞** | 正常讀取 (阻塞直到有資料) | 不阻塞，回傳零值與 `ok=false` |
| **寫入 `ch<-`** | **永久阻塞** | 正常寫入 (阻塞直到被讀走) | **Panic (send on closed channel)** |
| **關閉 `close()`** | **Panic** | 正常關閉 | **Panic (close of closed channel)** |

---

## 2. 避免死鎖 (Deadlock)

- **無緩衝通道**：發送方與接收方必須同時準備好，否則會永久阻塞。
- **緩衝通道**：發送方僅在緩衝區滿時阻塞；接收方僅在緩衝區空時阻塞。
- **單一 Goroutine 死鎖**：嚴禁在同一個 Goroutine 中對無緩衝通道進行同步讀寫。
- **建議**：使用 `select` 搭配 `time.After` 或 `context` 實作超時機制，避免無限期等待。

---

## 3. 關閉規則與優雅退出

- **發送方關閉原則**：**永遠只在發送方關閉 Channel**。
- **判斷關閉**：使用 `v, ok := <-ch`，當 `ok` 為 `false` 時代表 Channel 已關閉且資料已讀完。
- **資源洩漏**：如果接收方提前退出，而發送方仍在發送且無人接收，會導致發送方的 Goroutine 永久阻塞，造成記憶體洩漏。
- **For Range**：如果 Channel 不關閉，`for range` 迴圈會一直阻塞在讀取處！

---

## 4. 單向 Channel (One-way Channels)

為了提高型別安全，可以限制 Channel 的方向：

- **`chan<- T`**：唯寫 (Send-only)。常用於函式參數，確保該函式只能發送。
- **`<-chan T`**：唯讀 (Receive-only)。常用於回傳值，確保呼叫者只能讀取。

```go
func producer(ch chan<- int) {
    ch <- 1
}

func consumer(ch <-chan int) {
    val := <-ch
    fmt.Println(val)
}
```

---

## 5. Select 多路複用 (Multiplexing)

`select` 是處理多個 Channel 的利器，類似於 I/O 的 `switch`。

- **非阻塞讀寫**：加入 `default` 分支可實現非阻塞操作。
- **公平性**：當多個 `case` 同時就緒時，`select` 會**隨機**選擇一個執行，防止單一 Channel 飢餓。

```go
select {
case msg1 := <-ch1:
    fmt.Println("received", msg1)
case ch2 <- "hi":
    fmt.Println("sent hi")
default:
    fmt.Println("no communication")
}
```

---

## 6. 其他進階技巧

- **nil channel 的妙用**：在 `select` 中將某個 `case` 的 Channel 設為 `nil`，可以動態地禁用該分支（因為讀寫 `nil channel` 會永久阻塞）。
- **Signal Channel**：使用 `chan struct{}` 作為信號傳遞，節省記憶體空間。
