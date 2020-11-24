## 组合模式

### 7.1. 模式动机

组合模式在树形结构的问题中，模糊了简单元素和复杂元素的概念。客户程序一个像处理简单元素一样来处理复杂元素。从而使得客户程序与复杂的元素的内部结构解耦。

### 7.2. 模式定义

Composite Pattern 组合模式，又叫部分整体模式。用于把一组相似的对象当做一个单一的对象。

### 7.3. 代码实现

```
package composite

import "fmt"

type Component interface {
   Parent() Component
   SetParent(Component)
   Name() string
   SetName(string)
   AddChild(Component)
   Print(string)
}

const (
   LeafNode = iota
   CompositeNode
)

func NewComponent(kind int, name string) Component {
   var c Component
   switch kind {
   case LeafNode:
      c = NewLeaf()
   case CompositeNode:
      c = NewComposite()
   }

   c.SetName(name)
   return c
}

type component struct {
   parent Component
   name   string
}

func (c *component) Parent() Component {
   return c.parent
}

func (c *component) SetParent(parent Component) {
   c.parent = parent
}

func (c *component) Name() string {
   return c.name
}

func (c *component) SetName(name string) {
   c.name = name
}

func (c *component) AddChild(Component) {}

func (c *component) Print(string) {}

type Leaf struct {
   component
}

func NewLeaf() *Leaf {
   return &Leaf{}
}

func (c *Leaf) Print(pre string) {
   fmt.Printf("%s-%s\n", pre, c.Name())
}

type Composite struct {
   component
   childs []Component
}

func NewComposite() *Composite {
   return &Composite{
      childs: make([]Component, 0),
   }
}

func (c *Composite) AddChild(child Component) {
   child.SetParent(c)
   c.childs = append(c.childs, child)
}

func (c *Composite) Print(pre string) {
   fmt.Printf("%s+%s\n", pre, c.Name())
   pre += " "
   for _, comp := range c.childs {
      comp.Print(pre)
   }
}
```

```
package composite

func ExampleComposite() {
   root := NewComponent(CompositeNode, "root")
   c1 := NewComponent(CompositeNode, "c1")
   c2 := NewComponent(CompositeNode, "c2")
   c3 := NewComponent(CompositeNode, "c3")

   l1 := NewComponent(LeafNode, "l1")
   l2 := NewComponent(LeafNode, "l2")
   l3 := NewComponent(LeafNode, "l3")

   root.AddChild(c1)
   root.AddChild(c2)
   c1.AddChild(c3)
   c1.AddChild(l1)
   c2.AddChild(l2)
   c2.AddChild(l3)

   root.Print("")
   // Output:
   // +root
   //  +c1
   //   +c3
   //   -l1
   //  +c2
   //   -l2
   //   -l3
}
```