# lec 5: go memory model
## 介绍
Go 语言内存模型规定了共享变量的情况下所能做出的一些保证，这些约定是确保使用者在使用同步特性的 api 时能让程序以期望的方式运行，并强调任何不使用这些 api 情况下的并发的操作都是未定义的行为
## Happens Before
`Happens Before` 描述了两个操作指令之间先后关系，如果事件 e1 发生在 e2 之前，就可以说 e1 happens before e2。这种关系是可以级联的，如果 e2 happens before e3，那就可以推导出 e1 happens before e3。
首先有一条基础规则：在同一 goroutine 中，happens before 与代码的声明顺序一致。但是在多个goroutine 之间的代码是没有 Happens Before 的保证的，需要使用 `sync` 包的 api 来达成目的
## 使用 sync 包来达成 Happens Before 关系
这里简单的举个例子，具体规则看官网文档即可
### Locks
golang 里关于锁有两个数据结构可以用，sync.Mutex 和 sync.RWMutex
对任意一个 sync.Mutex/sync.RWMutex 变量 l 来说，第 n 次 `l.Unlock()` happens before 第 m 次 `l.Lock()` （n < m）
```
var l sync.Mutex
var a string

func f() {
	a = “hello, world” // action 1
	l.Unlock() // action 2
}

func main() {
	l.Lock() // action 3
	go f()
	l.Lock() // action 4
	print(a) // action 5
}
```
在这个例子当中，act1 happens before act2。而 act2 是第一次解锁，act4 是第二次加锁，根据规则，act2 happens before act4。根据级联关系可以推导出，act1 happens before act5。如果去除了锁，输出结果是不被保证的，即使你用 time.Sleep 让 goroutine f 先执行也是不可靠的。
