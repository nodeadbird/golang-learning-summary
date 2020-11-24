## 建造者模式

### 4.1. 模式动机

无论是在现实世界中还是在软件系统中，都存在一些复杂的对象，它们拥有多个组成部分，如汽车，它包括车轮、方向盘、发送机等各种部件。而对于大多数用户而言，无须知道这些部件的装配细节，也几乎不会使用单独某个部件，而是使用一辆完整的汽车，可以通过建造者模式对其进行设计与描述，建造者模式可以将部件和其组装过程分开，一步一步创建一个复杂的对象。用户只需要指定复杂对象的类型就可以得到该对象，而无须知道其内部的具体构造细节。

在软件开发中，也存在大量类似汽车一样的复杂对象，它们拥有一系列成员属性，这些成员属性中有些是引用类型的成员对象。而且在这些复杂对象中，还可能存在一些限制条件，如某些属性没有赋值则复杂对象不能作为一个完整的产品使用；有些属性的赋值必须按照某个顺序，一个属性没有赋值之前，另一个属性可能无法赋值等。

复杂对象相当于一辆有待建造的汽车，而对象的属性相当于汽车的部件，建造产品的过程就相当于组合部件的过程。由于组合部件的过程很复杂，因此，这些部件的组合过程往往被“外部化”到一个称作建造者的对象里，建造者返还给客户端的是一个已经建造完毕的完整产品对象，而用户无须关心该对象所包含的属性以及它们的组装方式，这就是建造者模式的模式动机。

### 4.2. 模式定义

建造者模式(Builder Pattern)：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

建造者模式是一步一步创建一个复杂的对象，它允许用户只通过指定复杂对象的类型和内容就可以构建它们，用户不需要知道内部的具体构建细节。建造者模式属于对象创建型模式。根据中文翻译的不同，建造者模式又可以称为生成器模式。

### 4.3. 模式结构

建造者模式包含如下角色：

- Builder：抽象建造者
- ConcreteBuilder：具体建造者
- Director：指挥者
- Product：产品角色

![../_images/Builder.jpg](./images/Builder.jpg)

### 4.4. 代码分析

```
package builder

//Builder 是生成器接口
type Builder interface {
   Part1()
   Part2()
   Part3()
}

type Director struct {
   builder Builder
}

// NewDirector ...
func NewDirector(builder Builder) *Director {
   return &Director{
      builder: builder,
   }
}

//Construct Product
func (d *Director) Construct() {
   d.builder.Part1()
   d.builder.Part2()
   d.builder.Part3()
}

type Builder1 struct {
   result string
}

func (b *Builder1) Part1() {
   b.result += "1"
}

func (b *Builder1) Part2() {
   b.result += "2"
}

func (b *Builder1) Part3() {
   b.result += "3"
}

func (b *Builder1) GetResult() string {
   return b.result
}

type Builder2 struct {
   result int
}

func (b *Builder2) Part1() {
   b.result += 1
}

func (b *Builder2) Part2() {
   b.result += 2
}

func (b *Builder2) Part3() {
   b.result += 3
}

func (b *Builder2) GetResult() int {
   return b.result
}
```

```
package builder

import "testing"

func TestBuilder1(t *testing.T) {
   builder := &Builder1{}
   director := NewDirector(builder)
   director.Construct()
   res := builder.GetResult()
   if res != "123" {
      t.Fatalf("Builder1 fail expect 123 acture %s", res)
   }
}

func TestBuilder2(t *testing.T) {
   builder := &Builder2{}
   director := NewDirector(builder)
   director.Construct()
   res := builder.GetResult()
   if res != 6 {
      t.Fatalf("Builder2 fail expect 6 acture %d", res)
   }
}
```

