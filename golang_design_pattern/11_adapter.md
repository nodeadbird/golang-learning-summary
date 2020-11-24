## 适配器模式

### 1.1. 模式动机

- 在软件开发中采用类似于电源适配器的设计和编码技巧被称为适配器模式。
- 通常情况下，客户端可以通过目标类的接口访问它所提供的服务。有时，现有的类可以满足客户类的功能需要，但是它所提供的接口不一定是客户类所期望的，这可能是因为现有类中方法名与目标类中定义的方法名不一致等原因所导致的。
- 在这种情况下，现有的接口需要转化为客户类期望的接口，这样保证了对现有类的重用。如果不进行这样的转化，客户类就不能利用现有类所提供的功能，适配器模式可以完成这样的转化。
- 在适配器模式中可以定义一个包装类，包装不兼容接口的对象，这个包装类指的就是适配器(Adapter)，它所包装的对象就是适配者(Adaptee)，即被适配的类。
- 适配器提供客户类需要的接口，适配器的实现就是把客户类的请求转化为对适配者的相应接口的调用。也就是说：当客户类调用适配器的方法时，在适配器类的内部将调用适配者类的方法，而这个过程对客户类是透明的，客户类并不直接访问适配者类。因此，适配器可以使由于接口不兼容而不能交互的类可以一起工作。这就是适配器模式的模式动机。

### 1.2. 模式定义

适配器模式(Adapter Pattern) ：将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

### 1.3. 模式结构

适配器模式包含如下角色：

- Target：目标抽象类
- Adapter：适配器类
- Adaptee：适配者类
- Client：客户类

适配器模式有对象适配器和类适配器两种实现：

对象适配器：

![../_images/Adapter.jpg](./images/Adapter.jpg)

类适配器：

![../_images/Adapter_classModel.jpg](./images/Adapter_classModel.jpg)

### 1.4. 代码分析

```
package adapter

//Target 是适配的目标接口
type Target interface {
   Request() string
}


//Adaptee 是被适配的目标接口
type Adaptee interface {
   SpecificRequest() string
}

//NewAdaptee 是被适配接口的工厂函数
func NewAdaptee() Adaptee {
   return &adapteeImpl{}
}



//AdapteeImpl 是被适配的目标类
type adapteeImpl struct{}

//SpecificRequest 是目标类的一个方法
func (*adapteeImpl) SpecificRequest() string {
   return "adaptee method"
}



//Adapter 是转换Adaptee为Target接口的适配器
type adapter struct {
   Adaptee
}

//NewAdapter 是Adapter的工厂函数
func NewAdapter(adaptee Adaptee) Target {
   return &adapter{
      Adaptee: adaptee,
   }
}

//Request 实现Target接口
func (a *adapter) Request() string {
   return a.SpecificRequest()
}
```

```
package adapter

import "testing"

var expect = "adaptee method"

func TestAdapter(t *testing.T) {
   adaptee := NewAdaptee()
   target := NewAdapter(adaptee)
   res := target.Request()
   if res != expect {
      t.Fatalf("expect: %s, actual: %s", expect, res)
   }
}
```