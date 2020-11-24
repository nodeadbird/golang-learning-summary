## 职责链模式

## 11.1. 模式动机

为了避免请求者和发送者耦合在一起，让多个对象都有可能接收数据，我们将这些接受者对象连城一个链，并且沿着这条链传递请求。直到有对象处理这个请求为止。

### 优点

- 降低耦合度
- 简化接收对象，增加和减少请求的处理很容易
- 通过改变链内的成员或者调动他们的顺序，可以动态新增和删除责任。

### 缺点

- 不能保证请求一定被处理
- 调试代码时不方便，不容易观察运行时的特征，排查错误不方便。

## 11.2. 模式定义

Chain of Responsibility Pattern为请求创建一个接受者对象的链，这样可以使得请求和发送者解耦。。

## 11.5. 代码分析

```
package chain

import "fmt"

type Manager interface {
   HaveRight(money int) bool
   HandleFeeRequest(name string, money int) bool
}

type RequestChain struct {
   Manager
   successor *RequestChain
}

func (r *RequestChain) SetSuccessor(m *RequestChain) {
   r.successor = m
}

func (r *RequestChain) HandleFeeRequest(name string, money int) bool {
   if r.Manager.HaveRight(money) {
      return r.Manager.HandleFeeRequest(name, money)
   }
   if r.successor != nil {
      return r.successor.HandleFeeRequest(name, money)
   }
   return false
}

func (r *RequestChain) HaveRight(money int) bool {
   return true
}

type ProjectManager struct{}

func NewProjectManagerChain() *RequestChain {
   return &RequestChain{
      Manager: &ProjectManager{},
   }
}

func (*ProjectManager) HaveRight(money int) bool {
   return money < 500
}

func (*ProjectManager) HandleFeeRequest(name string, money int) bool {
   if name == "bob" {
      fmt.Printf("Project manager permit %s %d fee request\n", name, money)
      return true
   }
   fmt.Printf("Project manager don't permit %s %d fee request\n", name, money)
   return false
}

type DepManager struct{}

func NewDepManagerChain() *RequestChain {
   return &RequestChain{
      Manager: &DepManager{},
   }
}

func (*DepManager) HaveRight(money int) bool {
   return money < 5000
}

func (*DepManager) HandleFeeRequest(name string, money int) bool {
   if name == "tom" {
      fmt.Printf("Dep manager permit %s %d fee request\n", name, money)
      return true
   }
   fmt.Printf("Dep manager don't permit %s %d fee request\n", name, money)
   return false
}

type GeneralManager struct{}

func NewGeneralManagerChain() *RequestChain {
   return &RequestChain{
      Manager: &GeneralManager{},
   }
}

func (*GeneralManager) HaveRight(money int) bool {
   return true
}

func (*GeneralManager) HandleFeeRequest(name string, money int) bool {
   if name == "ada" {
      fmt.Printf("General manager permit %s %d fee request\n", name, money)
      return true
   }
   fmt.Printf("General manager don't permit %s %d fee request\n", name, money)
   return false
}
```

```
package chain

func ExampleChain() {
   c1 := NewProjectManagerChain()
   c2 := NewDepManagerChain()
   c3 := NewGeneralManagerChain()

   c1.SetSuccessor(c2)
   c2.SetSuccessor(c3)

   var c Manager = c1

   c.HandleFeeRequest("bob", 400)
   c.HandleFeeRequest("tom", 1400)
   c.HandleFeeRequest("ada", 10000)
   c.HandleFeeRequest("floar", 400)
   // Output:
   // Project manager permit bob 400 fee request
   // Dep manager permit tom 1400 fee request
   // General manager permit ada 10000 fee request
   // Project manager don't permit floar 400 fee request

}
```