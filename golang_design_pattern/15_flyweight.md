## 享元模式

### 5.1. 模式动机

面向对象技术可以很好地解决一些灵活性或可扩展性问题，但在很多情况下需要在系统中增加类和对象的个数。当对象数量太多时，将导致运行代价过高，带来性能下降等问题。

- 享元模式正是为解决这一类问题而诞生的。享元模式通过共享技术实现相同或相似对象的重用。
- 在享元模式中可以共享的相同内容称为内部状态(IntrinsicState)，而那些需要外部环境来设置的不能共享的内容称为外部状态(Extrinsic State)，由于区分了内部状态和外部状态，因此可以通过设置不同的外部状态使得相同的对象可以具有一些不同的特征，而相同的内部状态是可以共享的。
- 在享元模式中通常会出现工厂模式，需要创建一个享元工厂来负责维护一个享元池(Flyweight Pool)用于存储具有相同内部状态的享元对象。
- 在享元模式中共享的是享元对象的内部状态，外部状态需要通过环境来设置。在实际使用中，能够共享的内部状态是有限的，因此享元对象一般都设计为较小的对象，它所包含的内部状态较少，这种对象也称为细粒度对象。享元模式的目的就是使用共享技术来实现大量细粒度对象的复用。

### 5.2. 模式定义

享元模式(Flyweight Pattern)：运用共享技术有效地支持大量细粒度对象的复用。系统只使用少量的对象，而这些对象都很相似，状态变化很小，可以实现对象的多次复用。由于享元模式要求能够共享的对象必须是细粒度对象，因此它又称为轻量级模式，它是一种对象结构型模式。

### 5.3. 模式结构

享元模式包含如下角色：

- Flyweight: 抽象享元类
- ConcreteFlyweight: 具体享元类
- UnsharedConcreteFlyweight: 非共享具体享元类
- FlyweightFactory: 享元工厂类

![../_images/Flyweight.jpg](./images/Flyweight.jpg)

### 5.3. 代码分析

```
package flyweight

import "fmt"

type ImageFlyweightFactory struct {
   maps map[string]*ImageFlyweight
}

var imageFactory *ImageFlyweightFactory

func GetImageFlyweightFactory() *ImageFlyweightFactory {
   if imageFactory == nil {
      imageFactory = &ImageFlyweightFactory{
         maps: make(map[string]*ImageFlyweight),
      }
   }
   return imageFactory
}

func (f *ImageFlyweightFactory) Get(filename string) *ImageFlyweight {
   image := f.maps[filename]
   if image == nil {
      image = NewImageFlyweight(filename)
      f.maps[filename] = image
   }

   return image
}

type ImageFlyweight struct {
   data string
}

func NewImageFlyweight(filename string) *ImageFlyweight {
   // Load image file
   data := fmt.Sprintf("image data %s", filename)
   return &ImageFlyweight{
      data: data,
   }
}

func (i *ImageFlyweight) Data() string {
   return i.data
}

type ImageViewer struct {
   *ImageFlyweight
}

func NewImageViewer(filename string) *ImageViewer {
   image := GetImageFlyweightFactory().Get(filename)
   return &ImageViewer{
      ImageFlyweight: image,
   }
}

func (i *ImageViewer) Display() {
   fmt.Printf("Display: %s\n", i.Data())
}
```

```
package flyweight

import "testing"

func ExampleFlyweight() {
   viewer := NewImageViewer("image1.png")
   viewer.Display()
   // Output:
   // Display: image data image1.png
}

func TestFlyweight(t *testing.T) {
   viewer1 := NewImageViewer("image1.png")
   viewer2 := NewImageViewer("image1.png")

   if viewer1.ImageFlyweight != viewer2.ImageFlyweight {
      t.Fail()
   }
}
```