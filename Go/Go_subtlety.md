Go中的微妙知识点汇编

# 1、Go语法篇

## 1.1、for := range

[Go规范](https://golang.google.cn/ref/spec#For_statements)中“For statements with `range` clause”明确指出，range表达式为数组（或切片）、字符串时, “:=”左侧出现的迭代变量的值至多有2个，分别是下标和对应的值。如果只出现1个，则视为下标。

> for index := range []int{0xa, 0xb} {
>
> ​	// index为切片的下标，即0、1
>
> }

当range表达式为map时，“:=”左侧出现的迭代变量的值至多有2个，分别是key和值。如果只出现1个变量，则视为key。

> for key := range map[string]bool{"key1":true} {
>
> ​	// key为key1
>
> }

当range表达式为channel时，“:=”左侧出现的迭代变量只有1个，即为通道的元素。

## 1.2、switch表达式

在[Go规范](https://golang.google.cn/ref/spec#Switch_statements)中，switch有switch表达式和type switch 2种用法。我想说的是switch表达式的注意事项。首先，计算switch表达式得到的值类型和case分支的值的类型必须一样。其次，Go规定缺失switch表达式暗示总是true，即布尔类型。这意味着case分支表达式的值类型须是布尔类型。最后，switch表达式若没有匹配的case分支（即便没有default），则程序会跳出switch语句，继续执行。

## 1.3、接口类型的零值

在Go中，一个接口的零值为nil，当且仅当它的类型和值都为nil。

> var i interface{}
>
> fmt.Printf("%T %v %", i, i, i == nil)    // "nil" "nil" "true"
>
> i = ([]int)(nil)
>
> fmt.Printf("%T %v %", i, i, i == nil)    // "[]int" "[]" "false"

这个知识点在《The Go Programming Language》书中有很到位的阐述。