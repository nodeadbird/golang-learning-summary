## 命令模式

### 1.1. 模式动机

在软件设计中，我们经常需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道被请求的操作是哪个，我们只需在程序运行时指定具体的请求接收者即可，此时，可以使用命令模式来进行设计，使得请求发送者与请求接收者消除彼此之间的耦合，让对象之间的调用关系更加灵活。

命令模式可以对发送者和接收者完全解耦，发送者与接收者之间没有直接引用关系，发送请求的对象只需要知道如何发送请求，而不必知道如何完成请求。这就是命令模式的模式动机。

### 1.2. 模式定义

命令模式(Command Pattern)：将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作(Action)模式或事务(Transaction)模式。

### 1.3. 模式结构

命令模式包含如下角色：

- Command: 抽象命令类

- ConcreteCommand: 具体命令类

- Invoker: 调用者

- Receiver: 接收者

- Client:客户类

  ![../_images/Command.jpg](./images/Command.jpg)

### 1.4. 代码分析

命令模式本质是把某个对象的方法调用封装到对象中，方便传递、存储、调用。

示例中把主板单中的启动(start)方法和重启(reboot)方法封装为命令对象，再传递到主机(box)对象中。于两个按钮进行绑定：

- 第一个机箱(box1)设置按钮1(buttion1) 为开机按钮2(buttion2)为重启。
- 第二个机箱(box1)设置按钮2(buttion2) 为开机按钮1(buttion1)为重启。

从而得到配置灵活性。

除了配置灵活外，使用命令模式还可以用作：

- 批处理
- 任务队列
- undo, redo

等把具体命令封装到对象中使用的场合

```
package command

import "fmt"

type Command interface {
   Execute()
}

type StartCommand struct {
   mb *MotherBoard
}

func NewStartCommand(mb *MotherBoard) *StartCommand {
   return &StartCommand{
      mb: mb,
   }
}

func (c *StartCommand) Execute() {
   c.mb.Start()
}

type RebootCommand struct {
   mb *MotherBoard
}

func NewRebootCommand(mb *MotherBoard) *RebootCommand {
   return &RebootCommand{
      mb: mb,
   }
}

func (c *RebootCommand) Execute() {
   c.mb.Reboot()
}

type MotherBoard struct{}

func (*MotherBoard) Start() {
   fmt.Print("system starting\n")
}

func (*MotherBoard) Reboot() {
   fmt.Print("system rebooting\n")
}

type Box struct {
   buttion1 Command
   buttion2 Command
}

func NewBox(buttion1, buttion2 Command) *Box {
   return &Box{
      buttion1: buttion1,
      buttion2: buttion2,
   }
}

func (b *Box) PressButtion1() {
   b.buttion1.Execute()
}

func (b *Box) PressButtion2() {
   b.buttion2.Execute()
}
```

```
package command

func ExampleCommand() {
   mb := &MotherBoard{}
   startCommand := NewStartCommand(mb)
   rebootCommand := NewRebootCommand(mb)

   box1 := NewBox(startCommand, rebootCommand)
   box1.PressButtion1()
   box1.PressButtion2()

   box2 := NewBox(rebootCommand, startCommand)
   box2.PressButtion1()
   box2.PressButtion2()
   // Output:
   // system starting
   // system rebooting
   // system rebooting
   // system starting
}
```

