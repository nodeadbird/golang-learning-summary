## 外观模式

### 4.1. 模式动机

### 4.2. 模式定义

外观模式(Facade Pattern)：外部与一个子系统的通信必须通过一个统一的外观对象进行，为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。外观模式又称为门面模式，它是一种对象结构型模式。

### 4.3. 模式结构

外观模式包含如下角色：

- Facade: 外观角色
- SubSystem:子系统角色

![../_images/Facade.jpg](./images/Facade.jpg)

### 4.4. 代码分析

```
package facade

import "fmt"

func NewAPI() API {
   return &apiImpl{
      a: NewAModuleAPI(),
      b: NewBModuleAPI(),
   }
}

//API is facade interface of facade package
type API interface {
   Test() string
}

//facade implement
type apiImpl struct {
   a AModuleAPI
   b BModuleAPI
}

func (a *apiImpl) Test() string {
   aRet := a.a.TestA()
   bRet := a.b.TestB()
   return fmt.Sprintf("%s\n%s", aRet, bRet)
}

//NewAModuleAPI return new AModuleAPI
func NewAModuleAPI() AModuleAPI {
   return &aModuleImpl{}
}

//AModuleAPI ...
type AModuleAPI interface {
   TestA() string
}

type aModuleImpl struct{}

func (*aModuleImpl) TestA() string {
   return "A module running"
}

//NewBModuleAPI return new BModuleAPI
func NewBModuleAPI() BModuleAPI {
   return &bModuleImpl{}
}

//BModuleAPI ...
type BModuleAPI interface {
   TestB() string
}

type bModuleImpl struct{}

func (*bModuleImpl) TestB() string {
   return "B module running"
}
```

```
package facade

import "testing"

var expect = "A module running\nB module running"

// TestFacadeAPI ...
func TestFacadeAPI(t *testing.T) {
   api := NewAPI()
   ret := api.Test()
   if ret != expect {
      t.Fatalf("expect %s, return %s", expect, ret)
   }
}
```

