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

- **基本關閉原則**：**只在發送方（Sender）關閉 Channel，且只在單個發送方時直接關閉**。嚴禁在接收方（Receiver）或多個發送方並行時直接呼叫 `close()`。
- **判斷關閉**：使用 `v, ok := <-ch`，當 `ok` 為 `false` 時代表 Channel 已關閉且資料已讀完。
- **資源洩漏**：如果接收方提前退出，而發送方仍在發送且無人接收，會導致發送方的 Goroutine 永久阻塞，造成記憶體洩漏。
- **For Range**：如果 Channel 不關閉，`for range` 迴圈會一直阻塞在讀取處！

### 🛠️ 多發送方/多接收方場景的優雅關閉模式

當有多個發送方時，直接在其中一個發送方關閉 Channel 會導致其他發送方在寫入時觸發 `panic: send on closed channel`。此時可採用以下幾種方案：

#### 方案 A：使用 `sync.Once` 確保只關閉一次
如果多個 goroutine 都有可能觸發關閉事件，可以使用 `sync.Once` 包裝 `close` 操作：

```go
type SafeChannel struct {
    ch   chan int
    once sync.Once
}

func (sc *SafeChannel) Close() {
    sc.once.Do(func() {
        close(sc.ch)
    })
}
```

#### 方案 B：使用協調通道 (Stop/Signal Channel) —— 推薦首選
引入一個額外的唯讀廣播通道 `stopCh (chan struct{})`。
1. **不關閉資料通道**，而是當接收方決定停止接收，或協調者決定退出時，**關閉 `stopCh`**。
2. 發送方透過 `select` 監聽 `stopCh`。當 `stopCh` 被關閉後，發送方立即停止寫入並退出。
3. 未被關閉的資料通道在沒有任何 goroutine 引用後，會被 Go 的垃圾回收器 (GC) 自動回收。

```go
func main() {
    dataCh := make(chan int, 100)
    stopCh := make(chan struct{})

    // N 個發送者
    for i := 0; i < 3; i++ {
        go func(id int) {
            for {
                select {
                case <-stopCh: // 收到停止訊號，退出
                    return
                case dataCh <- id:
                    time.Sleep(100 * time.Millisecond)
                }
            }
        }(i)
    }

    // 接收者在滿足某條件後決定停止
    go func() {
        for v := range dataCh {
            fmt.Println("Received:", v)
            if v == 2 { // 假設收到 2 就停止
                close(stopCh) // 廣播關閉訊號，所有發送者會安全退出
                return
            }
        }
    }()

    time.Sleep(2 * time.Second)
}
```

#### 方案 C：使用 `sync.WaitGroup` 協調關閉
如果必須關閉資料通道（例如下游使用 `for range` 讀取），且發送者數量已知：
1. 使用 `sync.WaitGroup` 追蹤發送者 Goroutine 的數量。
2. 每個發送者完成後呼叫 `wg.Done()`。
3. 另開一個協調者 Goroutine 等待 `wg.Wait()`，待所有發送者結束後，**由協調者負責關閉資料通道**。

```go
var wg sync.WaitGroup
dataCh := make(chan int)

// 啟動發送者
for i := 0; i < 3; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        dataCh <- 1
    }()
}

// 啟動一個協調者負責 close
go func() {
    wg.Wait()
    close(dataCh) // 安全關閉，因為所有發送者都已結束
}()
```


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
