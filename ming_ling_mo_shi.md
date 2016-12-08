# 命令模式
命令模式是我最爱的设计模式之一。在很多我写过的大型程序里，命令模式都随处可以。他可以让一些原来丑陋的代码变得十分整洁。然而对于这样一个优秀的设计的模式，GOF给出的解释却十分晦涩。

>  a request as an object, thereby letting users parameterize clients with different requests, queue or log requests, and support undoable operations.  
> 将请求压缩成对象，使用不同的请求实现客户端参数化、排队、日志等功能，并且支持撤销操作。

我想你也觉得这个句子很难懂吧。首先它没有说明白它想比喻的内容，一些词脱离了计算机领域就会有不同的意思，比如client可以指一个人，你的客户，而“人”显然是不能被“参数化”的。接下来，这个句子介绍了一些可能用到该设计模式的地方，但如果不是你亲自遇到，是很难理解的。所以我给了命令模式一个精炼的定义。
> A command is a reified method call  
> 一个命令(command)就是一个具体化的方法调用。

当然，由于精炼带来的过度简化，导致这个定义依然难以理解。让我来进一步解释一下，“具体化"可以理解为"make real"，也可以理解为让方法变成"first-class"。这两种解释的本质都是说将一些内容封装到一个对象中去，然后将对象作为参数传递给函数。所以我将“命令"解释成”具体化的方法调用“，也就是说将一个函数调用函数封装成一个对象。

这听起来有点像回调函数、first-class函数,函数指针、闭包、偏应用函数等概念，这些概念本质上都差不多，只是取决于你用的语言不同。GOF也在书后提到

> Commands are an object-oriented replacement for callbacks.  
> 命令模式以面向对象的方式取代了回调函数

用这句话去解释命令模式比GOF之前的定义更加贴切。但无论怎讲解释，这些定义依然是十分抽象的，还是让我们用一些实际的例子来理解命令模式吧。

## 输入配置
在游戏代码中，我们经常需要一堆代码来处理用户输入，包括按钮、键盘、鼠标等，这些代码将用户输入转换成游戏中有意义的行为。
![](command-buttons-one.png)
一个简单实现如下：
```cpp
void InputHandler::handleInput()
{
  if (isPressed(BUTTON_X)) jump();
  else if (isPressed(BUTTON_Y)) fireGun();
  else if (isPressed(BUTTON_A)) swapWeapon();
  else if (isPressed(BUTTON_B)) lurchIneffectively();
}
```
这个函数一般会在游戏中每帧调用一次，其作用也很容易理解。但是这段代码将按键和行为进行了硬编码，导致玩家无法在游戏中实现自定义按键。

为了实现自定义按键功能，我们将jump()和fireGun()这些直接的调用进行转变，使用对象来封装这些行为。进入：命令模式。

我们定义了一个基类表示一个可触发的游戏命令：
```cpp
class Command
{
public:
  virtual ~Command() {}
  virtual void execute() = 0;
};
```

