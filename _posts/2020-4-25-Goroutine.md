### 并发
1. 当一个函数创建为goroutine时，Go会将其视为一个独立的工作单元，这个单元会被调度到可用的逻辑处理器上执行
2. Go语言的并发同步模型来自一个叫作通信顺序进程(Communicating Sequential Process,CSP)的泛型，CSP是一种消息传递模型，通过使用通道(channel)在goroutine之间传递数据来传递消息
3. 如果希望让goroutine并行，必须使用多于一个逻辑处理器，但是要想真正实现并行的效果，程序需要运行在有多个物理处理器的机器上
4. Sync包中的`WaitGroup`是一个计数信号量，可以用来记录并维护运行的goroutine，如果`WaitGroup`的值大于0，`Wait`方法将会阻塞，使用`Done`方法则会将其值减一
5. 基于调度器的内部算法，一个正运行的goroutine在工作结束前，可以被停止并重新调度，这样做的目的是防止某个goroutine长时间占用逻辑处理器，当某个goroutine占用时间时间过长时调度器会停止当前正运行的goroutine，并给其他可运行的goroutine运行的机会
6. 通过`runtime`包中的`GOMAXPROCS`函数可以指定调度器可用的逻辑处理器的数量，而`NumCPU`函数返回可以使用的物理处理器数量
7. 同一时刻只能有一个goroutine对共享资源进行读和写操作，否则会产生竞争状态(race condition)，可以使用`-race`工具检测代码里的竞争状态。
