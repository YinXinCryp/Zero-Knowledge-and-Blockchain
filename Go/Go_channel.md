# Go Channel用法小结

# 1. channel声明

声明channel的基本形式：

```
var c chan T 
var c chan<- T  // send-only channel
var c <-chan T  // receive-only channel
```

`T`可以是内置的基础类型、复合类型（例如，`slice`，数组、`map`、`chan`、`func`、`interface{}`）、自定义类型（例如，`struct{}`）。`chan`是关键字。

## 2. 实例化

```
var c chan struct{} // 声明
c = make(chan struct{}) // 无缓冲通道
等价于 c = make(chan struct{}, 0) // 无缓冲通道
或者 c := make(chan struct{})
```

实例化需要用到`make`关键字。

实例化一个有缓冲通道：c := make(chan error, 1) // 1为通道缓冲的容量

## 3. 内置的3个函数

```
len(c)   // 通道缓冲中处于排队的元素个数
cap(c)   // 通道缓冲的容量
close(c) //  标明无更多值要发送。一般发送侧关闭。
```

例如：

```
c := make(chan error, 1)
fmt.Println(cap(c), len(c)) // "1"  "0"
```

## 4. API

1、接收

```
element <- c  // 通道变量在<-的右侧
<- c  // 不关心通道中的元素
```

2、发送

```
c <- element  // 通道变量在<-的左侧
```

3、遍历

（1）基本形式

```
for e := range c {}  // 迭代值e是通道c上已发送的值，直到通道关闭。
```

（2）变体

```
for range c { } // 将通道中（缓冲的，如果是带缓冲的通道）倒腾空（而且不在意通道中的元素）
```

遍历用法实例

```
ctx, cancelFn := context.WithTimeout(context.Background(), time.Second)
defer cancelFn()
for range ctx.Done() { 	
	fmt.Println("timeout") // 这句话不会被打印出来。因为for range仅仅作用在已关闭的无缓冲通道上。
}
```

修复方案：

```
ctx, cancelFn := context.WithTimeout(context.Background(), time.Second)
defer cancelFn()
<- ctx.Done()  // 因为关闭通道后，接收操作会返回类型的零值。
for range ctx.Done() { 	
	fmt.Println("timeout") // 这句话不会被打印出来。因为for range仅仅作用在已关闭的无缓冲通道上。
}
```

例子：

```
errC := make(chan error, 1)
errC <- nil
close(errC)
for range errC{
	fmt.Println("within for range")  // 正常打印
}
```

4、select语句

例子：

```
ctx, cancelFn := context.WithTimeout(context.Background(), time.Second)
defer cancelFn()
select {
case <-ctx.Done(): 	
	fmt.Println("timeout") // 正常打印，但不如<-ctx.Done来得简洁。
}
```

## 5. 操作表

​                                      nil channel                           1                       2                  3              4

for range c                      blocked                        blocked           blocked           ok

c <- element                    panic                           blocked                ok              panic       

 <- c                                   panic                           blocked            blocked        panic        ok

close(c)                             panic                               ok                      ok             panic        panic

1:initialized and unbuffered channel

2: initialized and buffered channel 

3: closed (initialized and unbuffered/buffered ) channel

4：receive-only channel

5：send-only channel

## 6. 如何优雅的关闭通道？

send(goroutine)      recv(goroutine)

   1    ------   c    ------     1      // sender(goroutine) just closes c

   n    ------   c    ------     1      // only a goroutine can close c and only one time

   1    ------   c    ------     n     //  sender(goroutine) just closes c

   n    ------   c    ------     n     //  only a goroutine can close c and only one time