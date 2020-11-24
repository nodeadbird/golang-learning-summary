## 观察者模式

## 3.1. 模式动机

建立一种对象与对象之间的依赖关系，一个对象发生改变时将自动通知其他对象，其他对象将相应做出反应。在此，发生改变的对象称为观察目标，而被通知的对象称为观察者，一个观察目标可以对应多个观察者，而且这些观察者之间没有相互联系，可以根据需要增加和删除观察者，使得系统更易于扩展，这就是观察者模式的模式动机。

## 3.2. 模式定义

观察者模式(Observer Pattern)：定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。观察者模式又叫做发布-订阅（Publish/Subscribe）模式、模型-视图（Model/View）模式、源-监听器（Source/Listener）模式或从属者（Dependents）模式。

观察者模式是一种对象行为型模式。

## 3.3. 模式结构

观察者模式包含如下角色：

- Subject: 目标
- ConcreteSubject: 具体目标
- Observer: 观察者
- ConcreteObserver: 具体观察者

![../_images/Obeserver.jpg](https://design-patterns.readthedocs.io/zh_CN/latest/_images/Obeserver.jpg)

## 3.4. 时序图

![../_images/seq_Obeserver.jpg](https://design-patterns.readthedocs.io/zh_CN/latest/_images/seq_Obeserver.jpg)

## 3.5. 代码分析

```
package observer

import "fmt"

type Subject struct {
   observers []Observer
   context   string
}

func NewSubject() *Subject {
   return &Subject{
      observers: make([]Observer, 0),
   }
}

func (s *Subject) Attach(o Observer) {
   s.observers = append(s.observers, o)
}

func (s *Subject) notify() {
   for _, o := range s.observers {
      o.Update(s)
   }
}

func (s *Subject) UpdateContext(context string) {
   s.context = context
   s.notify()
}

type Observer interface {
   Update(*Subject)
}

type Reader struct {
   name string
}

func NewReader(name string) *Reader {
   return &Reader{
      name: name,
   }
}

func (r *Reader) Update(s *Subject) {
   fmt.Printf("%s receive %s\n", r.name, s.context)
}
```

```
package observer

func ExampleObserver() {
   subject := NewSubject()
   reader1 := NewReader("reader1")
   reader2 := NewReader("reader2")
   reader3 := NewReader("reader3")
   subject.Attach(reader1)
   subject.Attach(reader2)
   subject.Attach(reader3)

   subject.UpdateContext("observer mode")
   // Output:
   // reader1 receive observer mode
   // reader2 receive observer mode
   // reader3 receive observer mode
}
```