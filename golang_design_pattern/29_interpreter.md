## 解释器模式

## 9.1. 模式动机

给定一个语言，定义他的文法，并定义一个解释器，这个解释器使用该标识来解释语言中的句子。如果一种特定类型的问题发生的频率足够高，那么就值得将该问题的各个实例表述为一个简单语言中的句子，这样就可以构建一个解释器，解释器通过解析这些句子来解决该问题。

### 优点

- 可拓展性高，灵活
- 易于实现简单的文法

### 缺点

- 可使用场景少
- 对于复杂的文法较难维护
- 解释器模式会引起类膨胀
- 解释器模式采用递归调用的方法

## 9.2. 模式定义

解释器模式实现了一个表达式的接口，该接口解释一个特定的上下文。这种模式被用在SQL语句处理，符号解释引擎等。

## 9.5. 代码分析

```
package interpreter

import (
   "strconv"
   "strings"
)

type Node interface {
   Interpret() int
}

type ValNode struct {
   val int
}

func (n *ValNode) Interpret() int {
   return n.val
}

type AddNode struct {
   left, right Node
}

func (n *AddNode) Interpret() int {
   return n.left.Interpret() + n.right.Interpret()
}

type MinNode struct {
   left, right Node
}

func (n *MinNode) Interpret() int {
   return n.left.Interpret() - n.right.Interpret()
}

type Parser struct {
   exp   []string
   index int
   prev  Node
}

func (p *Parser) Parse(exp string) {
   p.exp = strings.Split(exp, " ")

   for {
      if p.index >= len(p.exp) {
         return
      }
      switch p.exp[p.index] {
      case "+":
         p.prev = p.newAddNode()
      case "-":
         p.prev = p.newMinNode()
      default:
         p.prev = p.newValNode()
      }
   }
}

func (p *Parser) newAddNode() Node {
   p.index++
   return &AddNode{
      left:  p.prev,
      right: p.newValNode(),
   }
}

func (p *Parser) newMinNode() Node {
   p.index++
   return &MinNode{
      left:  p.prev,
      right: p.newValNode(),
   }
}

func (p *Parser) newValNode() Node {
   v, _ := strconv.Atoi(p.exp[p.index])
   p.index++
   return &ValNode{
      val: v,
   }
}

func (p *Parser) Result() Node {
   return p.prev
}
```

```
package interpreter

import "testing"

func TestInterpreter(t *testing.T) {
   p := &Parser{}
   p.Parse("1 + 2 + 3 - 4 + 5 - 6")
   res := p.Result().Interpret()
   expect := 1
   if res != expect {
      t.Fatalf("expect %d got %d", expect, res)
   }
}
```