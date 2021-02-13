

2021/01/11，Go Team公布泛型草案《Type Parameters — Draft Design》，征求社区意见。[经过充分的讨论](https://github.com/golang/go/issues/43651)，该提案已被接受（accepted）。

《草案设计》阅读笔记

Just as regular parameters have types, types parameters have meta-types, also knowns as constraits. 

any is a valid constrait, meaning that any type is permitted.

在浏览社区开发者的“聊天记录”，我整理了以下要点。

1、On type parameters
Type parameters are a new kind of type, but still just a type.
Type parameters introduce abstractions, and needless abstraction makes code hard to read.
Type parameters are enclosed using square brackets.


2、 On any type
any is predeclared as type any interface{}.any just means interface{}, nothing more, nothing less. 

But a type parameter list is instantiated (through substitution) at compile time; 
an ordinary parameter list is assigned to (via function invocation) at run time.



IMO(in my opinion)
With great power comes great responsibility(能力越大，责任越大)