# Go 的一个 CAS 操作使用场景

大概一年前，曾经遇到这么一个问题：
程序中有 N个并发执行的routine，都会向一个size 为 n 的 channel 里面写入数据，这 N 个 routine 有比较高的并发度，同时负载也比较大，所以不希望在写入数据的时候卡住，因此使用了这样的代码。

```
if len(c) < n {
    c <- something // 写入
}
```

原本意义是，保证一定能够写入，防止 worker routine 卡住。但实际运行过程中发生了routine 卡在 channel 写入处的现象，原因非常简单：

> 多个 routine 同时判断`len(c)`没有满，并且同时进入了写入 channel 的代码，当 channel 满了，处理如果不及时，那么后写入的routine 便会阻塞在此。

使用一个`sync.Mutex`把检查长度和写入 channel 的代码保护起来当然可以解决，但是由于认为这个 Mutex 可能影响性能，实际上使用的是一个比较**low**的办法解决。

```
const (
    _CHAN_SIZE  = 10
    _GUARD_SIZE = 10
)

var c chan interface{} = make(_CHAN_SIZE + _GUARD_SIZE) // 额外分配了一块保护的空间。

func write(val interface{}) {
    if len(c) < _CHAN_SIZE {
        c <- val
    }
}
```

在并发执行的多个 routine R1,R2...Rn 的中，同一时间只允许唯一一个 routine 执行某一个操作，并且其他 routine 需要非阻塞的知道自己无权操作并返回的时候，可以使用 CAS 操作。

对于这些 worker routine来说，情况大概是这样的：

> ***『弱弱的瞥一眼那个位置(操作)，没人占着咱就占，其他人占着咱也不等，直接走人』\***

比较优雅的方式，是使用go标准库里面的 `atomic.CompareAndSwap` 这一族函数。

```
// CompareAndSwapInt64 executes the compare-and-swap operation for an int64 value.
func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)
...
```

这些函数功能很简单，当给定的地址的值和 `old` 相等的时候，设置为新值，同时返回`true`，否则返回`false`。
该函数为原子操作。

维基百科上的描述：
[比较并交换(compare and swap, CAS)](https://zh.wikipedia.org/wiki/比较并交换)

于是上面的代码可以这么写：

```
func writeMsgWithCASCheck(val interface{}) {
    if atomic.CompareAndSwapInt64(&flag, 0, 1) {
        if len(c) < _CHAN_SIZE {
            c <- val
            atomic.StoreInt64(&obj.flag, 0)
            return nil
        }
        atomic.StoreInt64(&obj.flag, 0)
    }
}
```

如果要保证一定写入进去的话，可以在 atomic外面再套一个 for:

```
func writeMsgWithCASCheck(val interface{}) {
    for {
        if atomic.CompareAndSwapInt64(&flag, 0, 1) {
            if len(c) < _CHAN_SIZE {
                ...
        }
    }
}
```

但这样的效果就和直接卡在 `c <- val`一样，还 占满了 cpu（处于忙等状态）。

针对这种情况我写了一个简单的测试程序：

```
$ go run cas.go
R(0)+1 R(0)+1 R(0)+1 R(0)+1 R(0)+1 R(0)+1 R(0)+1 R(2)+1 R(3)+1 R(1)+1 R(0)+1 R(1)+1 R(2)+1 R(3)+1 Chan overflow, len: 13.
quit.
$ go run cas.go cas
R(0)+1 R(0)+1 R(0)+1 R(0)+1 R(0)+1 R(0)+1 R(0)+1 R(3)+1 R(1)+1 R(2)+1 R(1)+1 R(0)+1 R(3)+1 R(2)+1 R(1)+1 R(3)+1 R(3)+1 R(3)+1 R(3)+1 R(1)+1 R(2)+1 R(2)+1 R(2)+1 R(3)+1 R(1)+1 R(2)+1 R(3)+1 R(1)+1 R(1)+1 R(2)+1 R(1)+1 R(2)+1 <nil>
quit.
```

开4个 routine 不停写入的情况下还是很容易出现写入超过预期size 的情况的。

完整代码如下`cas.go`：

```
package main

import (
    "errors"
    "fmt"
    "os"
    "sync/atomic"
    "time"
)

const (
    _CHAN_SIZE  = 10
    _GUARD_SIZE = 10

    _TEST_CNT = 32
)

type Obj struct {
    flag int64
    c    chan interface{}
}

func (obj *Obj) readLoop() error {
    counter := _TEST_CNT
    for {
        time.Sleep(5 * time.Millisecond)
        if len(obj.c) > _CHAN_SIZE {
            return errors.New(fmt.Sprintf("Chan overflow, len: %v.", len(obj.c)))
        } else if len(obj.c) > 0 {
            <-obj.c
            counter--
        }
        if counter <= 0 {
            return nil
        }
    }
}

func (obj *Obj) writeMsg(idx int, v interface{}) (err error) {
    for {
        if len(obj.c) < _CHAN_SIZE {
            obj.c <- v
            fmt.Printf("R(%v)+1 ", idx)
            return nil
        }
    }
}

func (obj *Obj) writeMsgWithCASCheck(idx int, v interface{}) (err error) {
    for {
        if atomic.CompareAndSwapInt64(&obj.flag, 0, 1) {
            if len(obj.c) < _CHAN_SIZE {
                obj.c <- v
                atomic.StoreInt64(&obj.flag, 0)
                fmt.Printf("R(%v)+1 ", idx)
                return nil
            } else {
                atomic.StoreInt64(&obj.flag, 0)
            }
        }
    }

    return nil
}

func main() {
    useCAS := false
    if len(os.Args) > 1 && os.Args[1] == "cas" {
        useCAS = true
    }
    routineCnt := 4
    tryCnt := _TEST_CNT / routineCnt
    var obj = &Obj{c: make(chan interface{}, _CHAN_SIZE+_GUARD_SIZE)}

    for idx := 0; idx < routineCnt; idx++ {
        go func(nameIdx int) {
            for tryIdx := 0; tryIdx < tryCnt; tryIdx++ {
                if useCAS {
                    obj.writeMsgWithCASCheck(nameIdx, nil)
                } else {
                    obj.writeMsg(nameIdx, nil)
                }
            }
        }(idx)
    }

    // fmt.Println(casObj.readLoop())
    fmt.Println(obj.readLoop())
    fmt.Println("quit.")
}
```