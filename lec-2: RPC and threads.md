# lec 2: RPC and threads
## 前言
本节主要围绕多线程并发和 RPC 通信延伸出来的各种复杂性来进行讲述

## 线程
多线程并发让程序的处理能力上升了一个数量级，同时也打开了潘多拉魔盒，各种千奇百怪的并发问题接踵而来，越是复杂的系统，越是防不胜防。传统的多线程应用通常是一个线程只做一件事，很快这也到了瓶颈，基于回调的单线程异步 io 也流行起来，一个线程就可以
同时做 N 件事，再和多线程结合起来，编程难度又上了一个台阶。万幸各种语言都抽象了合理的编程模型，解放大家的心智。

### Go 里的并发
go 有个独特的概念：goroutine。一个比线程更小的单位，一个 goroutine 只做一件事，一个程序的 goroutine 可以有几十万个。

### 共享数据
在并发场景下最常见的就是多个线程操作同一个数据，常用的解决方案是加锁或保证只有一个线程能够接触对应数据（即避免共享数据，用传递数据来代替）
加锁的副作用是可能会出现死锁的情况。

### 关于 RPC
RPC 是一种对网络通信的封装，RPC 框架会让使用者在感知上与调用本地函数相近，解放心智负担。具体的协议可以有很多种，比较流行的 grpc 就是对 http2 的封装，这门课程中使用的是 go 自带的线程间通信。