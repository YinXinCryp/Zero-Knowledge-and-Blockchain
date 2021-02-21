# 1、泛型设计草案

2021/01/11，Go Team公布泛型草案《Type Parameters — Draft Design》，征求社区意见。[经过充分的讨论](https://github.com/golang/go/issues/43651)，该提案已被接受（accepted）。

《草案设计》阅读笔记

Just as regular parameters have types, types parameters have meta-types, also knowns as constraits. 

any is a valid constrait, meaning that any type is permitted.

# 2、issue

在浏览社区开发者的“聊天记录”，我整理了以下要点。

1、On type parameters
Type parameters are a new kind of type, but still just a type.
Type parameters introduce abstractions, and needless abstraction makes code hard to read.
Type parameters are enclosed using square brackets.

2、 On any type
any is predeclared as type any interface{}.any just means interface{}, nothing more, nothing less. 

But a type parameter list is instantiated (through substitution) at compile time; 
an ordinary parameter list is assigned to (via function invocation) at run time.

# 3、GopherCon 2020上的泛型报告

Go语言创始3巨头之一Robert Griesemer在全球Go大会上对最新的泛型设计进行了讲解，[收听大会视频回放，请点我](https://www.bilibili.com/video/BV12h411f7Hp?from=search&seid=14650038572868709880)。目前的泛型设计是Ian Lance Taylor（Go Team的大神）联合Robert提出的。Robert在大会上直言，他是2018年才加入其中的。整个[幻灯片](./slides/Typing_Generic_Go_GoCon2020_Robert_Griesemer.pdf)内容组织得非常有条理，由浅入深，将泛型设计中最基础的内容都覆盖到了（例如，type parameters list，type parameters declaration and usage，实例化，constrait、any constrait）。下面是我做的一些笔记：

> 实例化
> 1、用type argument替换掉全部签名中的type parameters。
> 2、验证每个type argument满足type parameter约束；
>
> 函数调用
> 1、使用实例化后的签名，验证一般的argument（如往常一样）。
>
> 除了泛型函数，还可以有泛型类型。例如，泛型接口、泛型结构体。

此次大会上讲述的设计和Go Team接受的设计草案非常一致，如果看设计草案迷失方向，可以看看幻灯片。



IMO(in my opinion)
With great power comes great responsibility(能力越大，责任越大)