## 原型模式

### 6.1. 模式动机

原型模式用于创建重复的对象。当一个类在创建时开销比较大时(比如大量数据准备，数据库连接)，我们可以缓存该对象，当下一次调用时，返回该对象的克隆。

### 6.2. 模式定义

用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。通过实现克隆clone()操作，快速的生成和原型对象一样的实例。

### 6.3. 代码分析

```
package prototype

//Cloneable 是原型对象需要实现的接口
type Cloneable interface {
   Clone() Cloneable
}

type PrototypeManager struct {
   prototypes map[string]Cloneable
}

func NewPrototypeManager() *PrototypeManager {
   return &PrototypeManager{
      prototypes: make(map[string]Cloneable),
   }
}

func (p *PrototypeManager) Get(name string) Cloneable {
   return p.prototypes[name]
}

func (p *PrototypeManager) Set(name string, prototype Cloneable) {
   p.prototypes[name] = prototype
}
```

```
package prototype

import "testing"

var manager *PrototypeManager

type Type1 struct {
   name string
}

func (t *Type1) Clone() Cloneable {
   tc := *t
   return &tc
}

type Type2 struct {
   name string
}

func (t *Type2) Clone() Cloneable {
   tc := *t
   return &tc
}

func TestClone(t *testing.T) {
   t1 := manager.Get("t1")

   t2 := t1.Clone()

   if t1 == t2 {
      t.Fatal("error! get clone not working")
   }
}

func TestCloneFromManager(t *testing.T) {
   c := manager.Get("t1").Clone()

   t1 := c.(*Type1)
   if t1.name != "type1" {
      t.Fatal("error")
   }

}

func init() {
   manager = NewPrototypeManager()

   t1 := &Type1{
      name: "type1",
   }
   manager.Set("t1", t1)
}
```