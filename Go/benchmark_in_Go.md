> 没有度量就没有优化。
>
> when in doubt, benchmark it.  —— [2]

# 1、什么是Go的benchmark？

see [Go标准库benchmark简介](https://golang.google.cn/pkg/testing/#hdr-Benchmarks)。

## 1.1 Go内核的benchmark测试case

Go Team收集了一些社区产品级项目的[基准测试case列表](https://github.com/golang/go/wiki/Benchmarks)，度量代码修改对Go内核（编译器，运行时，垃圾收集器和标准库）的影响。其中的项目包括go-ethereum。例如[P256密钥生成算法](https://github.com/ethereum/go-ethereum/blob/8f03e3b107c0f7a39de31a9e7deb658431a937ac/crypto/ecies/ecies_test.go#L166)、[P256共享密钥生成算法](https://github.com/ethereum/go-ethereum/blob/8f03e3b107c0f7a39de31a9e7deb658431a937ac/crypto/ecies/ecies_test.go#L175)的基准测试。

# 2、为什么要做benchmark测试？

在一个Go应用中，我们可以用[pprof](https://github.com/google/pprof)工具来识别性能瓶颈。它可以用来剖析内存分配，CPU耗费，堆内存，goroutine阻塞和锁竞争。程序为什么慢？首先，我们可以用pprof来分析哪个函数最耗费CPU；锁定目标函数后，我们制定优化方案。如何检验优化效果呢？一个立杆见影的方法是为这个函数写benchmark测试。通过对优化前后的函数进行benchmark结果对照，我们可以看出优化效果，进而指导我们的下一步工作。

# 3、benchmark测试结果解读

第一列是benchmark函数名称；函数名后会跟着分配的处理器数量（可通过-cpu标志设置），若处理器数量大于1。当-cpu 1时，函数名无后缀，否则后面会跟着处理器数量。

第二列是for循环运行的迭代次数；

第三列是单次迭代消耗的时间（以纳秒计）。该值越低，效果越好。

*第四列是单次迭代分配的字节数量。该值越低，内存使用效果越好，

*第五列是单次迭代的内存分配次数。该值越低，效果越好。

> *要看到内存使用效果，需要在go test后面加-benchmen标志。

# 4、如何运行benchmark测试？

go test -bench=基准测试函数名  

go test -cpu 2 -bench=基准测试函数名  

go test -benchmen  -bench=基准测试函数名  

go test -bench=.

# 5、如何写benchmark测试？

## 5.1 待测程序

> // pubsub.go 
>
> // mainly from [1]
>
> type chans []chan string
>
> type Pubsub struct{
>
> ​	mu sync.RWMutex
>
> ​	subs map[string]chans
>
> }
>
> func NewPubsub() *Pubsub{
>
> ​	ps := &Pubsub{}
>
> ​	ps.subs = make(map[string]chans)
>
> }
>
> func (ps *Pubsub) Subscribe(topic string) <-chan string {
>
> ​	ps.mu.Lock()
>
> ​	defer ps.mu.Unlock()
>
> ​	ch := make(chan string, 1)
>
> ​	ps.subs[topics] = append(ps.subs[topic], ch)
>
> ​	return ch
>
> }
>
> func (ps *Pubsub) Publish(topic string, msg string) <-chan string {
>
> ​	ps.mu.Lock()
>
> ​	defer ps.mu.Unlock()
>
> ​	for  _, ch := range ps.subs[topics] {
>
> ​		ch <- msg
>
> ​	}
>
> }
>
> func (ps *Pubsub) AsyncPublish(topic string, msg string) <-chan string {
>
> ​	ps.mu.Lock()
>
> ​	defer ps.mu.Unlock()
>
> ​	for  _, ch := range ps.subs[topics] {
>
> ​		go func(ch chan string) {
>
> ​			ch <- msg
>
> ​		}(ch)
>
> ​	}
>
> }
>
> func (ps *Pubsub) SelectPublish(topic string, msg string) <-chan string {
>
> ​	ps.mu.Lock()
>
> ​	defer ps.mu.Unlock()
>
> ​	for  _, ch := range ps.subs[topics] {
>
> ​		select {
>
> ​		case ch <- msg:
>
> ​		default:
>
> ​		}
>
> ​	}
>
> }

## 5.2 benchmark测试文件

> // pubsub_test.go
>
> var pubsub *Pubsub
>
> func init(){
>
> ​	pubsub = NewPubsub()
>
> ​	ch1 := pubsub.Subscribe("topic1")
>
> ​	listener := func(name string, ch <- chan string) {
>
> ​		for {	
>
> ​			select {
>
> ​			case <- ch:
>
> ​			default:
>
> ​			}
>
> ​		}
>
> ​	}
>
> }
>
> // go test -bench=BenchmarkPublish
>
> // go test -benchmen -bench=BenchmarkPublish
>
> func BenchmarkPublish(b *testing.B) {
>
> ​	b.ResetTimer()
>
> ​	for n :=0; n < b.N; n++ {
>
> ​		pubsub.Publish("topic1", "BenchmarkPublish")
>
> ​	}
>
> }
>
> // go test -bench=BenchmarkAsyncPublish
>
> // go test -benchmen -bench=BenchmarkAsyncPublish
>
> func BenchmarkAsyncPublish(b *testing.B) {
>
> ​	b.ResetTimer()
>
> ​	for n :=0; n < b.N; n++ {
>
> ​		pubsub.AsyncPublish("topic1", "BenchmarkAsyncPublish")
>
> ​	}
>
> }
>
> // go test -bench=BenchmarkSelectPublish 测试单个函数，但只能获得前三列的数据。
>
> // run 'go test -benchmen -bench=BenchmarkSelectPublish' 可以查看内存使用
>
> func BenchmarkSelectPublish(b *testing.B) {
>
> ​	b.ResetTimer()
>
> ​	for n :=0; n < b.N; n++ {
>
> ​		pubsub.SelectPublish("topic1", "BenchmarkSelectPublish")
>
> ​	}
>
> }
>
> // 基准测试全部的函数：go test -benchmem -bench=.

# 6、参考文章

1、 [Application tuning in Go：benchmarks](https://brendanjryan.com/2018/01/15/go-benchmarks.html)

2、 [PubSub using channels in Go](https://eli.thegreenplace.net/2020/pubsub-using-channels-in-go/)

3、[Benchmarking Results in Go](https://mikenewswanger.com/posts/2018/benchmarking-in-go/)

4、[High Performance Go Workshop](https://dave.chenny.net/high-performance-go-workshop/dotgo-paris.html)