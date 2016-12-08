## 命令模式
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
> 当你有一个接口只有一个没有返回值的方法的时候，这是一个用命令模式的绝佳机会

然后我们再为不同的游戏行为定义不同的子类
```cpp
class JumpCommand : public Command
{
public:
  virtual void execute() { jump(); }
};

class FireCommand : public Command
{
public:
  virtual void execute() { fireGun(); }
};
// You get the idea...
```
在我们的InputHandler里，为每个按键都存了一个到命令的指针。
```cpp
class InputHandler
{
public:
  void handleInput();

  // Methods to bind commands...

private:
  Command* buttonX_;
  Command* buttonY_;
  Command* buttonA_;
  Command* buttonB_;
};
```
现在的输入部分是这样的
```cpp
void InputHandler::handleInput()
{
  if (isPressed(BUTTON_X)) buttonX_->execute();
  else if (isPressed(BUTTON_Y)) buttonY_->execute();
  else if (isPressed(BUTTON_A)) buttonA_->execute();
  else if (isPressed(BUTTON_B)) buttonB_->execute();
}
```
> 这里没有去检测指针是否为NULL，因为我们假设了所有的按键都是有对应行为的。如果想支持不做任何事情的行为，直接定义一个excute()方法为空的类，并将按键的指针指向该类即可。这种模式被称为“空对象“。

现在我们不再是直接的调用方法了，而是多了一层间接的映射。
![](command-buttons-two.png)

以上简短介绍了命令模式。如果你能看出该模式的好处，就接着看剩下的内容吧。

## 角色操作
我们刚才定义的类有一个很大的局限，我们假定了这些顶层的jump()、fireGun()等指令都能找到玩家角色，并对其进行操作。而这个假定限制了这些函数的参数对象，所以我们现在不要爱让这些函数去寻找对象，而是将对象作为参数传进去：
```cpp
class Command
{
public:
  virtual ~Command() {}
  virtual void execute(GameActor& actor) = 0;
};
```
`GameActor`指的是游戏中的一个角色，我们将其传给execute()，继承的类就可以直接操作传递进去的对象了，就像这样：
```cpp
class JumpCommand : public Command
{
public:
  virtual void execute(GameActor& actor)
  {
    actor.jump();
  }
};
```
现在，我们这个类可以让任意角色进行跳跃了。我们修改了handleInput(),让它返回命令。
```cpp
Command* InputHandler::handleInput()
{
  if (isPressed(BUTTON_X)) return buttonX_;
  if (isPressed(BUTTON_Y)) return buttonY_;
  if (isPressed(BUTTON_A)) return buttonA_;
  if (isPressed(BUTTON_B)) return buttonB_;

  // Nothing pressed, so do nothing.
  return NULL;
}
```
因为这里并没有actor参数，所以不能立即调用。这里也看到了命令模式的另一个好处，我们可以延迟到它执行时再调用。

接着，我们需要一些代码来接收命令，并且传入具体的角色。
```cpp
Command* command = inputHandler.handleInput();
if (command)
{
  command->execute(actor);
}
```
假设actor是一个玩家角色的引用，这段代码能按照玩家的输入做出对应的行为。而通过这增加的一层重定向以及actor参数，我们可以通过改变actor参数，从而控制场景中任意一个角色。

实际上，这个特性并不通用，但经常会出现一些类似的用例。目前我们只考虑了玩家控制的角色，但是对于其他的被AI控制的角色呢？我们也可以用相同的命令模式来连接AI引擎和角色，AI代码只要生成命令对象就可以了。

这里AI代码和命令之间的解耦也给了我们更大的灵活度。我们可以给不同的角色绑定不同的AI模块，或者为多样的行为混合AI。想具有一个更加好斗的对手？给他一个更加好斗的AI去生成命令就可以了。实际上，我们可以把AI直接挂在玩家角色上，这在Demo模式下，游戏需要自动演示时，是十分有用的。

通过将命令封装成first-class对象，我们从这种直接调用的方式中进行了解耦。作为替代，我们也可使用一个队列或者命令流。
![](command-stream.png)

一些代码(InputHandler或者AI)将命令放到流中，而另一部分代码(分发者或者actor)获取命令并且运行她们。通过中间的队列，我们解耦了生产者与消费者。

> 如果我们对这些命令进行序列化，我们就可以在网络上发送它们。我们可以拿到用户的输入，发送到另一台机器上，并且重现它，这是网络多人游戏的基础。

## 撤销和重做

## 类和函数


