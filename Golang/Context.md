# Context

## 功能

+ 超時和截止日期： 可以設置 Context 的超時或截止日期，當超過指定時間後，自動取消相應操作，避免程序長時間阻塞或等待。

+ 取消信號： 可以通過 Context 來告知各個 goroutine 停止工作，以防止不必要的工作或資源浪費。

+ 值傳遞: 可以使用 context.WithValue 在 Context 中存儲鍵值對，這些值可以跨多個 goroutine 傳遞。

## 範例

+ 超時與取消信號

```
func main() {
	// 創建一個父 Context
	ctx := context.Background()

	// 創建一個帶有超時的子 Context，設置超時時間為 2 秒
	ctx, cancel := context.WithTimeout(ctx, 2*time.Second)
	defer cancel()

	// 在 goroutine 中執行一個耗時操作
	go func() {
		select {
		case <-time.After(3 * time.Second):
			fmt.Println("耗時操作完成")
		case <-ctx.Done():
			fmt.Println("超時退出或被取消")
		}
	}()

	// 等待一段時間，模擬程序執行
	time.Sleep(5 * time.Second)
}
```

+ 傳遞值

```
func main() {
	// 創建一個父 Context
	parentCtx := context.Background()

	// 使用 WithValue 方法在父 Context 中設置一個 userID
	ctxWithValue := context.WithValue(parentCtx, userIDKey, "123456")

	// 在 goroutine 中讀取 Context 中的 userID
	go func() {
		// 從 Context 中讀取 userID
		userID, ok := ctxWithValue.Value(userIDKey).(string)
		if !ok {
			fmt.Println("無法讀取 userID")
			return
		}
		fmt.Println("從 Context 中讀取到的 userID:", userID)
	}()

	// 等待一段時間，模擬程序執行
	select {}
}
```