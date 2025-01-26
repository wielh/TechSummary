# Goroutine 與 channel

## 簡介

在 Go 中，Goroutine 是一种轻量级的并发执行单位，由 Go 运行时（goroutine scheduler）进行管理。Goroutine 可以看作是一种轻量级的线程，但相比传统的操作系统线程，Goroutine 的创建和调度开销更小，并且可以在单个进程内同时运行数千甚至数百万个 Goroutine，因此它们更适合于高并发的并发编程。

以下是 Goroutine 的一些特点和用法：
  + 并发执行：Goroutine 可以在程序中并发地执行任务，不会阻塞主程序的执行。通过 Goroutine，你可以将耗时的任务（如 I/O 操作、计算密集型任务等）异步地执行，从而提高程序的并发性能。
  + 轻量级：Goroutine 的创建和销毁开销很小，一个 Goroutine 的栈空间默认大小只有几 KB，因此可以轻松地创建大量的 Goroutine，而不会导致资源消耗过多。
  + 使用 go 关键字启动：要创建一个 Goroutine，只需在函数调用前使用 go 关键字即可。例如：go someFunction()。
  + 通信通过通道（Channel）：Goroutine 之间通常通过通道进行通信和同步。通道是一种并发安全的数据结构，可以在 Goroutine 之间传递数据，并且可以确保数据的安全传输和同步。
  + 调度由运行时管理：Goroutine 的调度由 Go 运行时自动管理，不需要程序员手动介入。Go 运行时会负责将 Goroutine 分配到可用的线程上执行，并在需要时进行调度切换。

## Channel

在 Go 语言中，通道（Channel）是用于在 Goroutine 之间进行通信和同步的重要机制。通道是一种类型安全且并发安全的数据结构，可以用来传递数据和控制 Goroutine 的执行流程。

+ 避免競爭條件: channel 是一個線程安全的數據傳遞方式，它內建了同步機制。

+ 簡化同步： 可以確保在多個 goroutine 之間的協作過程中，數據傳遞有序且同步，從而減少了使用共享內存時需要手動鎖定（例如使用 sync.Mutex）的麻煩

+ 避免死鎖和狀態錯誤： 使用共享地址進行同步時，若沒有適當的鎖機制，可能會導致死鎖或其他同步錯誤。channel 通常不會出現這類問題，因為它的設計本身就考慮了協調和同步。

## 範例程式碼

example1

```
m := 200
n := 10
token := make(chan int, m)
for i := 0; i < m; i++ {
	token <- i
}
close(token)

var wg sync.WaitGroup
for i := 0; i < n; i++ {
	wg.Add(1)
	go func() {
		defer wg.Done()
		for val := range token {
			fmt.Printf("Hello, I am goruntine %d\n", val)
			t := time.Duration(rand.Intn(1000)) * time.Millisecond
			time.Sleep(t)
		}
	}()
}
wg.Wait()
```

example2 
```
m := 200
n := 10

var wg sync.WaitGroup
token := make(chan (struct{}), n)
for i := 0; i < m; i++ {
	wg.Add(1)
	token <- struct{}{}
	go func(n int) {
		defer func() {
			wg.Done()
			<-token
		}()
		fmt.Printf("Hello, I am goruntine %d\n", n)
		t := time.Duration(rand.Intn(1000)) * time.Millisecond
		time.Sleep(t)
	}(i)
}
wg.Wait()
close(token)
```