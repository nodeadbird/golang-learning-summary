## 桥接模式

### 2.1. 模式动机

设想如果要绘制矩形、圆形、椭圆、正方形，我们至少需要4个形状类，但是如果绘制的图形需要具有不同的颜色，如红色、绿色、蓝色等，此时至少有如下两种设计方案：

- 第一种设计方案是为每一种形状都提供一套各种颜色的版本。
- 第二种设计方案是根据实际需要对形状和颜色进行组合

对于有两个变化维度（即两个变化的原因）的系统，采用方案二来进行设计系统中类的个数更少，且系统扩展更为方便。设计方案二即是桥接模式的应用。桥接模式将继承关系转换为关联关系，从而降低了类与类之间的耦合，减少了代码编写量。

### 2.2. 模式定义

桥接模式(Bridge Pattern)：将抽象部分与它的实现部分分离，使它们都可以独立地变化。它是一种对象结构型模式，又称为柄体(Handle and Body)模式或接口(Interface)模式。

### 2.3. 模式结构

桥接模式包含如下角色：

- Abstraction：抽象类
- RefinedAbstraction：扩充抽象类
- Implementor：实现类接口
- ConcreteImplementor：具体实现类

![../_images/Bridge.jpg](./images/Bridge.jpg)

### 2.4. 代码分析

```
package bridge

import "fmt"

type AbstractMessage interface {
   SendMessage(text, to string)
}

type MessageImplementer interface {
   Send(text, to string)
}

type MessageSMS struct{}

func ViaSMS() MessageImplementer {
   return &MessageSMS{}
}

func (*MessageSMS) Send(text, to string) {
   fmt.Printf("send %s to %s via SMS", text, to)
}

type MessageEmail struct{}

func ViaEmail() MessageImplementer {
   return &MessageEmail{}
}

func (*MessageEmail) Send(text, to string) {
   fmt.Printf("send %s to %s via Email", text, to)
}

type CommonMessage struct {
   method MessageImplementer
}

func NewCommonMessage(method MessageImplementer) *CommonMessage {
   return &CommonMessage{
      method: method,
   }
}

func (m *CommonMessage) SendMessage(text, to string) {
   m.method.Send(text, to)
}

type UrgencyMessage struct {
   method MessageImplementer
}

func NewUrgencyMessage(method MessageImplementer) *UrgencyMessage {
   return &UrgencyMessage{
      method: method,
   }
}

func (m *UrgencyMessage) SendMessage(text, to string) {
   m.method.Send(fmt.Sprintf("[Urgency] %s", text), to)
}
```

```
package bridge

func ExampleCommonSMS() {
   m := NewCommonMessage(ViaSMS())
   m.SendMessage("have a drink?", "bob")
   // Output:
   // send have a drink? to bob via SMS
}

func ExampleCommonEmail() {
   m := NewCommonMessage(ViaEmail())
   m.SendMessage("have a drink?", "bob")
   // Output:
   // send have a drink? to bob via Email
}

func ExampleUrgencySMS() {
   m := NewUrgencyMessage(ViaSMS())
   m.SendMessage("have a drink?", "bob")
   // Output:
   // send [Urgency] have a drink? to bob via SMS
}

func ExampleUrgencyEmail() {
   m := NewUrgencyMessage(ViaEmail())
   m.SendMessage("have a drink?", "bob")
   // Output:
   // send [Urgency] have a drink? to bob via Email
}
```

