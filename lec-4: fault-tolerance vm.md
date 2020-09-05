# lec 4: fault-tolerance vm
#6.824#
## 前言
本节讲了一个偏底层的虚拟机的主备方案，不同于一般的应用程序级的主备，这种基于虚拟机的主备是 cpu 指令级的，这意味着不管上面运行着什么样的程序，都能够有备份，代价就是效率不够高，毕竟所有的东西都要复制。
## 复制（replication）的两种方案
主备的核心要求就是两者的状态需要完全一致，也就是需要 primary 将状态复制到 backup 中。通常有两种方案。
一种是最简单的镜像拷贝（State transfer）。将 primary 的中的所有状态打成镜像传输给 backup。好处是简单的粗暴，写起来不用动脑子；坏处是局限性很大，一旦涉及到的数据很大，比如复制一个数据库的状态，那是 GB 甚至 TB 级的量，复制一次代价巨大。
另外一种是复制操作指令（Replicated state machine）。根据图灵计算机的原理，指定函数指定输入，那么输出也是一定的，所以只要保持初识状态一致（通过第一种方法），并保证后续的操作指令一式两份地传达过去，那么后续的状态也是一致的。这个方法涉及到的io 就小了很多，毕竟操作指令通常都是 KB 级的。
## 涉及到的问题
### 操作指令的确定性问题
有些指令并不是一致的，比如获取硬件的 id，获取当前的时间等，这些指令会因为物理环境的不同导致执行结果不一致。
解决方法是 primay 将这些指令的执行结果一并放到 log entry 中，发给 backup 进行重放。
### 如何确保 backup 与 primary 的绝对一致
primay 接收到一个请求并执行操作后会生成 response，这个 response 不会马上发回给 client，而是被 hypervisor 拦截，等相关请求的 log entry 发送到了 backup 的 hypervisor 上并收到 ack 之后，primary 的 hypervisor 才会把 response 正式地返回给 client。
 两种情况：
- 如果 primary 在收到 ack 之前就故障了，backup 的输出也会被丢弃，所以在 client 端表现是一致的，即未收到 response。
- 如果 primary 在输出后故障，backup 会重复发送 response，这对于业务可接受的，因为 tcp 会处理重复的数据包。对磁盘来说，写入也是幂等的
### 网络中断导致的脑裂问题
由于网络问题，primay 和 backup 无法通信，会一起尝试提升为 primary，这个时候就需要一个第三方决策者来决定哪个是 primay，parper 用的方法是使用 disk server 提供的 atomic test-and-set 测试，（这大概就是为什么 primary 和 backup 需要使用同一个磁盘的原因）.
## 磁盘的内存访问竞争
这块涉及到知识盲区了，看不太懂。以后有机会再研究。
