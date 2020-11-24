# 使用Golang实现的无锁队列，性能与Disruptor相当达到1400万/秒

前一久看到一篇文章美团[高性能队列——Disruptor](https://zhuanlan.zhihu.com/p/23863915)，时候自己琢磨了一下；经过反复修改，实现了一个相似的无锁队列EsQueue，该无锁队列相对Disruptor，而言少了队列数量属性quantity的CAP操作，因此性能杠杠的，在测试环境：windows10，Core(TM) i5-3320M CPU 2.6G, 8G 内存，go1.7.4，性能达到1360万/秒。现在把代码发布出来，请同行验证一下，代码如下：

源代码发布到[http://github.com/yireyun/go-queue](https://link.zhihu.com/?target=http%3A//github.com/yireyun/go-queue)

```go
// esQueue
package esQueue

import (
	"runtime"
	"sync/atomic"
)

type esCache struct {
	value interface{}
	mark  bool
}

// lock free queue
type EsQueue struct {
	capaciity uint32
	capMod    uint32
	putPos    uint32
	getPos    uint32
	cache     []esCache
}

func NewQueue(capaciity uint32) *EsQueue {
	q := new(EsQueue)
	q.capaciity = minQuantity(capaciity)
	q.capMod = q.capaciity - 1
	q.cache = make([]esCache, q.capaciity)
	return q
}

func (q *EsQueue) Capaciity() uint32 {
	return q.capaciity
}

func (q *EsQueue) Quantity() uint32 {
	var putPos, getPos uint32
	var quantity uint32
	getPos = q.getPos
	putPos = q.putPos

	if putPos >= getPos {
		quantity = putPos - getPos
	} else {
		quantity = q.capMod + putPos - getPos
	}

	return quantity
}

// put queue functions
func (q *EsQueue) Put(val interface{}) (ok bool, quantity uint32) {
	var putPos, putPosNew, getPos, posCnt uint32
	var cache *esCache
	capMod := q.capMod
	for {
		getPos = q.getPos
		putPos = q.putPos

		if putPos >= getPos {
			posCnt = putPos - getPos
		} else {
			posCnt = capMod + putPos - getPos
		}

		if posCnt >= capMod {
			runtime.Gosched()
			return false, posCnt
		}

		putPosNew = putPos + 1
		if atomic.CompareAndSwapUint32(&q.putPos, putPos, putPosNew) {
			break
		} else {
			runtime.Gosched()
		}
	}

	cache = &q.cache[putPosNew&capMod]

	for {
		if !cache.mark {
			cache.value = val
			cache.mark = true
			return true, posCnt + 1
		} else {
			runtime.Gosched()
		}
	}
}

// get queue functions
func (q *EsQueue) Get() (val interface{}, ok bool, quantity uint32) {
	var putPos, getPos, getPosNew, posCnt uint32
	var cache *esCache
	capMod := q.capMod
	for {
		putPos = q.putPos
		getPos = q.getPos

		if putPos >= getPos {
			posCnt = putPos - getPos
		} else {
			posCnt = capMod + putPos - getPos
		}

		if posCnt < 1 {
			runtime.Gosched()
			return nil, false, posCnt
		}

		getPosNew = getPos + 1
		if atomic.CompareAndSwapUint32(&q.getPos, getPos, getPosNew) {
			break
		} else {
			runtime.Gosched()
		}
	}

	cache = &q.cache[getPosNew&capMod]

	for {
		if cache.mark {
			val = cache.value
			cache.mark = false
			return val, true, posCnt - 1
		} else {
			runtime.Gosched()
		}
	}
}

// round 到最近的2的倍数
func minQuantity(v uint32) uint32 {
	v--
	v |= v >> 1
	v |= v >> 2
	v |= v >> 4
	v |= v >> 8
	v |= v >> 16
	v++
	return v
}
使用golang的Test测试结果 
Go:  1, Times:  1000000, use:  40.5283ms,  40ns/op
Go:  2, Times:  2000000, use: 109.0824ms,  54ns/op
Go:  3, Times:  3000000, use: 212.6447ms,  70ns/op
Go:  4, Times:  4000000, use: 290.6942ms,  72ns/op
Go:  5, Times:  5000000, use: 370.2461ms,  74ns/op
Go:  6, Times:  6000000, use:   436.29ms,  72ns/op
Go:  7, Times:  7000000, use: 518.3449ms,  74ns/op
Go:  8, Times:  8000000, use: 595.8935ms,  74ns/op
Go:  9, Times:  9000000, use: 681.9546ms,  75ns/op
Go: 10, Times:  1000000, use:  83.5561ms,  83ns/op
Go: 11, Times:  1100000, use:  90.5608ms,  82ns/op
Go: 12, Times:  1200000, use:  98.0624ms,  81ns/op
Go: 13, Times:  1300000, use: 105.5723ms,  81ns/op
Go: 14, Times:  1400000, use: 114.0762ms,  81ns/op
Go: 15, Times:  1500000, use: 122.5817ms,  81ns/op
Go: 16, Times:  1600000, use: 130.5921ms,  81ns/op
Go:Sum, Times: 54100000, use: 4.0006803s,  73ns/op
```