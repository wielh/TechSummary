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

## 範例程式碼

```
func worker(id int, jobs <-chan int, results chan<- int) {
	for job := range jobs {
		fmt.Printf("Worker %d started job %d\n", id, job)
		time.Sleep(time.Second)
		fmt.Printf("Worker %d finished job %d\n", id, job)
		results <- job * 2 
	}
}

func main() {
	jobs := make(chan int, 5)  
	results := make(chan int, 5) 

	for i := 1; i <= 3; i++ {
		go worker(i, jobs, results)
	}

	for j := 1; j <= 5; j++ {
		jobs <- j
	}
	close(jobs) 

	for a := 1; a <= 5; a++ {
		result := <-results
		fmt.Println("Result:", result)
	}
}

```
