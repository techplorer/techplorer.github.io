不管是哪种编程语言，并发编程的语义设计一定是编程语言本身中一个很大的特性。在目前主流的几款编程语言里，go语言的并发语义设计的是很精妙且简洁的，那么从本文开始的接下来几篇文章里，我将陆续把自己在go语言并发编程方面的一些理解记录下来。

在go语言里有两个基础的库对于并发编程是至关重要的：一个是waitgroup，用来做goroutine生命周期管理；另外一个则是context，用于做信号控制，这两个都是用于实现并发控制语义。

### WaitGroup

我们知道，与系统级别的线程不一样，go语言的goroutine是很轻量的，它的创建和销毁的副作用很小，所以我们可以在业务代码里边大量的创建、销毁goroutine。也就是说这里边涉及到goroutine的生命周期管理，而这也是waitgroup的作用。它的用法比较简单，主要就是几个接口的调用：Add/Done/Wait：

- Add用于设置等待goroutine的数量
- 每个goroutine在完成后调用Done，表示完成

- Wait用于阻塞，直到所有goroutine完成

```
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {

	var wg sync.WaitGroup

	wg.Add(2)

	go func() {
		defer wg.Done()
		time.Sleep(time.Second * 2)
		fmt.Println("worker1 done...")
	}()

	go func() {
		defer wg.Done()
		time.Sleep(time.Second * 3)
		fmt.Println("worker2 done...")
	}()

	wg.Wait()
	fmt.Println("all work has finished.")
}
```



### context

- 什么是context

首先来说一下context的语义，或者说为什么需要context？

context其实一种request-scoped语义，比方说，后端接受请求时，一般会开几个goroutine去做一些操作(比如用户校验、权限校验、查询数据库、还可能继续调用下游的rpc服务等等)，假如用户此时中断了请求或者请求超时了，如果后端没有提供一种cancelation机制，那么对于后端服务来说，这条请求链路上的所有操作都还在忙碌的状态，很明显的这会造成资源浪费，context就是为了解决这个问题的，它可以实现级联取消和超时控制。



- 超时控制与级联取消

比方说，我们的一个搜索业务：需要根据用户输入的关键字，去服务器里查找对应的web文本和video，但是后端的web文本搜索服务和video搜索服务存在不同的集群或者服务上，所以我们的业务代码就涉及到跨服务级别的请求，那么在这种情况下，我们的业务代码里边就要考虑到超时控制的逻辑。

借助context提供的语义，我们可以将这种超时策略向下传递，一旦主逻辑有了取消操作，那么整条链路上的所有请求全部取消：

```
// set requested timeout: 100ms
ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
defer cancel()

var wg sync.WaitGroup
wg.Add(2)

var results = make([]string, 2, 2)
// launch a goroutine to fetch web text by keyword
go func() {
    defer wg.Done()
    resp, err := fetchWeb(ctx, "go concurrency")
    if err != nil {
        log.Println(err)
        return
    }
    results = append(results, resp)
}()

// launch a gouroutine to fetch video by keyword
go func() {
    defer wg.Done()
    resp, err := fetchVideo(ctx, "go cocurrency")
    if err != nil {
        log.Println(err)
        return
    }
    results = append(results, resp)
}()

wg.Wait()
```

(完整的代码链接见https://gitee.com/techplorer/golp/blob/master/playgo/concurrency/context.go)



### 深入理解

- WaitGroup的原子语义

因为三个接口Done、Wait、Add会涉及到多协程的data race问题，那么WaitGroup是这么解决这个问题的呢？

答案是通过内存对齐和原子指令操作，先来看它的定义：

```
// A WaitGroup must not be copied after first use.
type WaitGroup struct {
	noCopy noCopy

	// 64-bit value: high 32 bits are counter, low 32 bits are waiter count.
	// 64-bit atomic operations require 64-bit alignment, but 32-bit
	// compilers do not ensure it. So we allocate 12 bytes and then use
	// the aligned 8 bytes in them as state, and the other 4 as storage
	// for the sema.
	state1 [3]uint32
}
```

内部由三部分组成：第一部分是32bits的counter，用于记录有多少未完成的协程数量；第二部分是32bits的waiter，用于记录有多少外部等待；最后一部分是32bits的信号量，所以加起来一共12个字节。

因为Add和Done是有可能多协程并发调用的，所以要求内部counter和waiter一定是要同步更新的，不允许出现couter更新但是waiter未更新的情况，这就是WaitGroup内部要求的同步语义。



再来看一下state的接口源码，看一下是如何实现这个同步语义的：

```
// state returns pointers to the state and sema fields stored within wg.state1.
func (wg *WaitGroup) state() (statep *uint64, semap *uint32) {
	if uintptr(unsafe.Pointer(&wg.state1))%8 == 0 {
		return (*uint64)(unsafe.Pointer(&wg.state1)), &wg.state1[2]
	} else {
		return (*uint64)(unsafe.Pointer(&wg.state1[1])), &wg.state1[0]
	}
}
```

state()内部会判断是64位系统还是32位系统，这个地方的逻辑是这样：如果是64位系统，那么WaitGroup的首地址一定是8字节对齐的，那么配合go语言提供的原子指令(LoadUint64/AddUint64)，即可解决同步问题；如果是32位系统，那么WaitGroup的首地址一定是4字节对齐，但是如果我们在首地址往后移动4字节的话，此时的内存地址就变成8字节对齐，最后借助go语言提供的原子指令，那么也可以解决同步问题。理解这个逻辑的话，就能看懂state的函数实现了。