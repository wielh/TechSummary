# channel 使用注意事項
+ 避免死鎖
    + 如果沒有 goroutine 接收數據，而發送方一直發送數據，程式會死鎖。
    + 如果沒有 goroutine 發送數據，而接收方一直等待，程式也會死鎖。
    + 緩衝通道（make(chan int, N)）允許傳送方不阻塞，直到緩衝區已滿。
    + 可以考慮使用 context.timeout 搭配 <-ctx.Done() 限時

+ 關閉 channel 的規則
    + 只有發送方可以關閉 channel，接收方不能關閉。
    + 關閉一個已經關閉的 channel 會引發運行時恐慌。
    + channel 關閉後，繼續從中接收不會阻塞。当 channel 为空时，會傳回零值。

+ select 關鍵字
    + select 是 channel 多路复用的利器，可同时监听多个 channel 的状态。
    + 注意添加 default 分支或是計時以避免永久阻塞。

+ 如果接收方退出，而发送方仍在发送数据，可能会导致资源泄露。

+ 如果 channel 不关闭，for range 会一直阻塞！

+ 避免使用 nil channel