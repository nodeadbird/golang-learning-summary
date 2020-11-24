## 装饰模式

## 3.1. 模式动机

一般有两种方式可以实现给一个类或对象增加行为：

- 继承机制，使用继承机制是给现有类添加功能的一种有效途径，通过继承一个现有类可以使得子类在拥有自身方法的同时还拥有父类的方法。但是这种方法是静态的，用户不能控制增加行为的方式和时机。
- 关联机制，即将一个类的对象嵌入另一个对象中，由另一个对象来决定是否调用嵌入对象的行为以便扩展自己的行为，我们称这个嵌入的对象为装饰器(Decorator)

装饰模式以对客户透明的方式动态地给一个对象附加上更多的责任，换言之，客户端并不会觉得对象在装饰前和装饰后有什么不同。装饰模式可以在不需要创造更多子类的情况下，将对象的功能加以扩展。这就是装饰模式的模式动机。

## 3.2. 模式定义

装饰模式(Decorator Pattern) ：动态地给一个对象增加一些额外的职责(Responsibility)，就增加对象功能来说，装饰模式比生成子类实现更为灵活。其别名也可以称为包装器(Wrapper)，与适配器模式的别名相同，但它们适用于不同的场合。根据翻译的不同，装饰模式也有人称之为“油漆工模式”，它是一种对象结构型模式。

## 3.3. 模式结构

装饰模式包含如下角色：

- Component: 抽象构件
- ConcreteComponent: 具体构件
- Decorator: 抽象装饰类
- ConcreteDecorator: 具体装饰类

![../_images/Decorator.jpg](./images/Decorator.jpg)

### 3.4. 代码分析

```
package decorator

type Component interface {
   Calc() int
}

type ConcreteComponent struct{}

func (*ConcreteComponent) Calc() int {
   return 0
}

type MulDecorator struct {
   Component
   num int
}

func WarpMulDecorator(c Component, num int) Component {
   return &MulDecorator{
      Component: c,
      num:       num,
   }
}

func (d *MulDecorator) Calc() int {
   return d.Component.Calc() * d.num
}

type AddDecorator struct {
   Component
   num int
}

func WarpAddDecorator(c Component, num int) Component {
   return &AddDecorator{
      Component: c,
      num:       num,
   }
}

func (d *AddDecorator) Calc() int {
   return d.Component.Calc() + d.num
}
```

```
package decorator

import "fmt"

func ExampleDecorator() {
   var c Component = &ConcreteComponent{}
   c = WarpAddDecorator(c, 10)
   c = WarpMulDecorator(c, 8)
   res := c.Calc()

   fmt.Printf("res %d\n", res)
   // Output:
   // res 80
}
```

