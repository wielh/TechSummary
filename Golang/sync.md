# sync 

## introduction
Go 的 sync 包提供了一些基本的同步原語，用於在多個 goroutine 之間協調操作，避免 race condition 等問題。

## sync.Mutex
互斥鎖，用於保護臨界區，確保只有一個 goroutine 能夠同時執行該區域的代碼。

```
var count int
var mu sync.Mutex

func increment() {
    mu.Lock()
    count++
    mu.Unlock()
}
```

## sync.RWMutex
功能：讀寫鎖，允許多個讀操作並行執行，但寫操作是互斥的。RLock 與 Lock 都會禁止修改值

```
var rwMu sync.RWMutex
var sharedData int

func readData() {
    rwMu.RLock()
    fmt.Println("Read:", sharedData)
    rwMu.RUnlock()
}

func writeData(value int) {
    rwMu.Lock()
    sharedData = value
    rwMu.Unlock()
}
```

## sync.WaitGroup
用於等待一組 goroutine 完成，通常用於協調多個 goroutine。

```
func worker(id int) {
    defer wg.Done()
    fmt.Printf("Worker %d starting\n", id)
    fmt.Printf("Worker %d done\n", id)
}

func main() {
    for i := 1; i <= 5; i++ {
        wg.Add(1)
        go worker(i)
    }
    wg.Wait()
    fmt.Println("All workers done")
}
```

## sync.Once
用於確保某段代碼只執行一次，無論是否有多個 goroutine 並行執行。

## sync/atomic 
提供對整數的原子操作，包括增減、加法、交換、加載和存儲等。
+ AddInt32 / AddInt64：對整數進行原子加法。
+ LoadInt32 / LoadInt64：原子加載整數的值。
+ StoreInt32 / StoreInt64：原子存儲整數的值。
+ SwapInt32 / SwapInt64：將整數設置為新值，返回舊值。
+ CompareAndSwapInt32 / CompareAndSwapInt64：原子比較並交換值（CAS 操作）。