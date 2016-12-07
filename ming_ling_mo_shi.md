# 命令模式

命令模式是我最爱的设计模式之一。在很多我写过的大型程序里，命令模式都随处可以。他可以让一些原来丑陋的代码变得十分整洁。然而对于一个这样一个优秀的设计的模式，GOF却给出了一个十分晦涩的解释。

>  a request as an object, thereby letting users parameterize clients with different requests, queue or log requests, and support undoable operations.

> 将请求压缩成对象，使用不同的请求实现客户端参数化、排队、日志等功能，并且支持撤销操作。

我想你也觉得这个句子很难懂吧。首先它没有说明白它想比喻的内容，一些词脱离了计算机领域就会有不同的意思，比如client可以指一个人，你的客户，而“人”显然是不能被“参数化”的。

接下来，这个句子介绍了一些可能用到该设计模式的地方，但如果不是你亲自遇到了这些情况，是很难理解的。所以我给了命令模式一个精炼的定义。
> 一个命令(command)就是一个具体化的方法调用。

当然，精炼也意味着难以理解的简化，所以这个解释对理解也并没有什么提升。让我来进一步解释一下，“具体化"表示"make real"，另一个解释是让一些东西变成"first-class"。


Reflection systems in some languages let you work with the types in your program imperatively at runtime. You can get an object that represents the class of some other object, and you can play with that to see what the type can do. In other words, reflection is a reified type system.
Both terms mean taking some concept and turning it into a piece of data — an object — that you can stick in a variable, pass to a function, etc. So by saying the Command pattern is a “reified method call”, what I mean is that it’s a method call wrapped in an object.

That sounds a lot like a “callback”, “first-class function”, “function pointer”, “closure”, or “partially applied function” depending on which language you’re coming from, and indeed those are all in the same ballpark. The Gang of Four later says:

Then, the rest of that sentence is just a list of stuff you could maybe possibly use the pattern for. Not very illuminating unless your use case happens to be in that list. My pithy tagline for the Command pattern is: