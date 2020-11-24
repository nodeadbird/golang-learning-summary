## 模板方法模式

## 7.1. 模式动机

- 某些类别的算法中，实做了相同的方法，造成代码的重复.
- 控制子类别必须遵守的一些事项

## 7.2. 模式定义

模板方法模式定义了一个算法的步骤，并允许子类别为一个或多个步骤提供其实践方式。让子类别在不改变算法架构的情况下，重新定义算法中的某些步骤.

## 7.3. 模式结构

模式包含如下角色：

- 抽象聚合类: 定义一个抽象的容器

- 具体聚合类: 实现上面的抽象类，作为一个容器，用来存放元素，等待迭代

- 抽象迭代器: 迭代器接口，每个容器下都有一个该迭代器接口的具体实现

- 具体迭代器: 根据不同的容器，需要定义不同的具体迭代器，定义了游标移动的具体实现

  ![./_images/template.jpg](./images/template.jpg)

## 7.5. 代码分析

```
package templatemethod

import "fmt"

type Downloader interface {
   Download(uri string)
}

type template struct {
   implement
   uri string
}

type implement interface {
   download()
   save()
}

func newTemplate(impl implement) *template {
   return &template{
      implement: impl,
   }
}

func (t *template) Download(uri string) {
   t.uri = uri
   fmt.Print("prepare downloading\n")
   t.implement.download()
   t.implement.save()
   fmt.Print("finish downloading\n")
}

func (t *template) save() {
   fmt.Print("default save\n")
}

type HTTPDownloader struct {
   *template
}

func NewHTTPDownloader() Downloader {
   downloader := &HTTPDownloader{}
   template := newTemplate(downloader)
   downloader.template = template
   return downloader
}

func (d *HTTPDownloader) download() {
   fmt.Printf("download %s via http\n", d.uri)
}

func (*HTTPDownloader) save() {
   fmt.Printf("http save\n")
}

type FTPDownloader struct {
   *template
}

func NewFTPDownloader() Downloader {
   downloader := &FTPDownloader{}
   template := newTemplate(downloader)
   downloader.template = template
   return downloader
}

func (d *FTPDownloader) download() {
   fmt.Printf("download %s via ftp\n", d.uri)
}
```

```
package templatemethod

func ExampleHTTPDownloader() {
   var downloader Downloader = NewHTTPDownloader()

   downloader.Download("http://example.com/abc.zip")
   // Output:
   // prepare downloading
   // download http://example.com/abc.zip via http
   // http save
   // finish downloading
}

func ExampleFTPDownloader() {
   var downloader Downloader = NewFTPDownloader()

   downloader.Download("ftp://example.com/abc.zip")
   // Output:
   // prepare downloading
   // download ftp://example.com/abc.zip via ftp
   // default save
   // finish downloading
}
```