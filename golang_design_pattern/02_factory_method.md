## 工厂方法模式

### 2.1. 模式动机

现在对该系统进行修改，不再设计一个按钮工厂类来统一负责所有产品的创建，而是将具体按钮的创建过程交给专门的工厂子类去完成，我们先定义一个抽象的按钮工厂类，再定义具体的工厂类来生成圆形按钮、矩形按钮、菱形按钮等，它们实现在抽象按钮工厂类中定义的方法。这种抽象化的结果使这种结构可以在不修改具体工厂类的情况下引进新的产品，如果出现新的按钮类型，只需要为这种新类型的按钮创建一个具体的工厂类就可以获得该新按钮的实例，这一特点无疑使得工厂方法模式具有超越简单工厂模式的优越性，更加符合“开闭原则”。

### 2.2. 模式定义

工厂方法模式(Factory Method Pattern)又称为工厂模式，也叫虚拟构造器(Virtual Constructor)模式或者多态工厂(Polymorphic Factory)模式，它属于类创建型模式。在工厂方法模式中，工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作延迟到工厂子类中完成，即通过工厂子类来确定究竟应该实例化哪一个具体产品类。

### 2.3. 模式结构

工厂方法模式包含如下角色：

- Product：抽象产品
- ConcreteProduct：具体产品
- Factory：抽象工厂
- ConcreteFactory：具体工厂

![./images/FactoryMethod.jpg](./images/FactoryMethod.jpg)

### 2.4. 代码分析

工厂方法模式使用子类的方式延迟生成对象到子类中实现。

Go中不存在继承 所以使用匿名组合来实现

```
package factorymethod

//Operator 是被封装的实际类接口
type Operator interface {
   SetA(int)
   SetB(int)
   Result() int
}

//OperatorBase 是Operator 接口实现的基类，封装公用方法
type OperatorBase struct {
   a, b int
}

//SetA 设置 A
func (o *OperatorBase) SetA(a int) {
   o.a = a
}

//SetB 设置 B
func (o *OperatorBase) SetB(b int) {
   o.b = b
}

//OperatorFactory 是工厂接口
type OperatorFactory interface {
   Create() Operator
}

//PlusOperatorFactory 是 PlusOperator 的工厂类
type PlusOperatorFactory struct{}

func (PlusOperatorFactory) Create() Operator {
   return &PlusOperator{
      OperatorBase: &OperatorBase{},
   }
}

//PlusOperator Operator 的实际加法实现
type PlusOperator struct {
   *OperatorBase
}

//Result 获取结果
func (o PlusOperator) Result() int {
   return o.a + o.b
}

//MinusOperatorFactory 是 MinusOperator 的工厂类
type MinusOperatorFactory struct{}

func (MinusOperatorFactory) Create() Operator {
   return &MinusOperator{
      OperatorBase: &OperatorBase{},
   }
}

//MinusOperator Operator 的实际减法实现
type MinusOperator struct {
   *OperatorBase
}

//Result 获取结果
func (o MinusOperator) Result() int {
   return o.a - o.b
}
```

```
package factorymethod

import "testing"

func compute(factory OperatorFactory, a, b int) int {
   op := factory.Create()
   op.SetA(a)
   op.SetB(b)
   return op.Result()
}

func TestOperator(t *testing.T) {
   var (
      factory OperatorFactory
   )

   factory = PlusOperatorFactory{}
   if compute(factory, 1, 2) != 3 {
      t.Fatal("error with factory method pattern")
   }

   factory = MinusOperatorFactory{}
   if compute(factory, 4, 2) != 2 {
      t.Fatal("error with factory method pattern")
   }
}
```

