## 状态模式

## 4.1. 模式动机

- 在很多情况下，一个对象的行为取决于一个或多个动态变化的属性，这样的属性叫做状态，这样的对象叫做有状态的(stateful)对象，这样的对象状态是从事先定义好的一系列值中取出的。当一个这样的对象与外部事件产生互动时，其内部状态就会改变，从而使得系统的行为也随之发生变化。
- 在UML中可以使用状态图来描述对象状态的变化。

## 4.2. 模式定义

状态模式(State Pattern) ：允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。其别名为状态对象(Objects for States)，状态模式是一种对象行为型模式。

## 4.3. 模式结构

状态模式包含如下角色：

- Context: 环境类
- State: 抽象状态类
- ConcreteState: 具体状态类

![../_images/State.jpg](https://design-patterns.readthedocs.io/zh_CN/latest/_images/State.jpg)

## 4.4. 时序图

![../_images/seq_State.jpg](https://design-patterns.readthedocs.io/zh_CN/latest/_images/seq_State.jpg)



## 4.5. 代码分析

```
package state

import "fmt"

type Week interface {
   Today()
   Next(*DayContext)
}

type DayContext struct {
   today Week
}

func NewDayContext() *DayContext {
   return &DayContext{
      today: &Sunday{},
   }
}

func (d *DayContext) Today() {
   d.today.Today()
}

func (d *DayContext) Next() {
   d.today.Next(d)
}

type Sunday struct{}

func (*Sunday) Today() {
   fmt.Printf("Sunday\n")
}

func (*Sunday) Next(ctx *DayContext) {
   ctx.today = &Monday{}
}

type Monday struct{}

func (*Monday) Today() {
   fmt.Printf("Monday\n")
}

func (*Monday) Next(ctx *DayContext) {
   ctx.today = &Tuesday{}
}

type Tuesday struct{}

func (*Tuesday) Today() {
   fmt.Printf("Tuesday\n")
}

func (*Tuesday) Next(ctx *DayContext) {
   ctx.today = &Wednesday{}
}

type Wednesday struct{}

func (*Wednesday) Today() {
   fmt.Printf("Wednesday\n")
}

func (*Wednesday) Next(ctx *DayContext) {
   ctx.today = &Thursday{}
}

type Thursday struct{}

func (*Thursday) Today() {
   fmt.Printf("Thursday\n")
}

func (*Thursday) Next(ctx *DayContext) {
   ctx.today = &Friday{}
}

type Friday struct{}

func (*Friday) Today() {
   fmt.Printf("Friday\n")
}

func (*Friday) Next(ctx *DayContext) {
   ctx.today = &Saturday{}
}

type Saturday struct{}

func (*Saturday) Today() {
   fmt.Printf("Saturday\n")
}

func (*Saturday) Next(ctx *DayContext) {
   ctx.today = &Sunday{}
}
```

```
package state

func ExampleWeek() {
   ctx := NewDayContext()
   todayAndNext := func() {
      ctx.Today()
      ctx.Next()
   }

   for i := 0; i < 8; i++ {
      todayAndNext()
   }
   // Output:
   // Sunday
   // Monday
   // Tuesday
   // Wednesday
   // Thursday
   // Friday
   // Saturday
   // Sunday
}
```