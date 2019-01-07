## groupcache源码分析（四）-- singleflight
singleflight.go文件中是singleflight模块的代码，这主要是进行相同访问的一个合并操作。也就是说，如果对于某个key的请求已经存在并且正在进行，则对该key的新的请求会堵塞在这里，等原来的请求结束后，将请求得到的结果同时返回给堵塞中的请求。

该部分就封装了一个接口：

```
func (g *Group) Do(key string, fn func() (interface{}, error)) (interface{}, error)
```
首先，先定义了下面两个结构体：

```go
//实际请求函数的封装结构体
// call is an in-flight or completed Do call
type call struct {
	wg  sync.WaitGroup
	//实际的请求函数
	val interface{}
	err error
}

//主要是用来组织已经存在的对某key的请求和对应的实际请求函数映射
// Group represents a class of work and forms a namespace in which
// units of work can be executed with duplicate suppression.
type Group struct {
	//用于对m上锁，保护m
	mu sync.Mutex       // protects m
	m  map[string]*call // lazily initialized
}
```

下面具体讲解一下这个函数。

该函数入参是一个key和一个实际请求函数，出参是一个接口类型和一个错误类型。

这里利用了go的锁机制，比如Metux、WaitGroup等。

```go
// Do executes and returns the results of the given function, making
// sure that only one execution is in-flight for a given key at a
// time. If a duplicate comes in, the duplicate caller waits for the
// original to complete and receives the same results.
func (g *Group) Do(key string, fn func() (interface{}, error)) (interface{}, error) {
	//有可能要修改m，所以先上锁进行保护
	g.mu.Lock()
	//如果m为nil，则初始化一个
	if g.m == nil {
		g.m = make(map[string]*call)
	}
	//如果m中存在对该key的请求，则该线程不会直接再次访问key，所以释放锁
	//然后堵塞等待已经存在的请求得到的结果
	if c, ok := g.m[key]; ok {
		//解锁
		g.mu.Unlock()
		//堵塞
		c.wg.Wait()
		//如果已经存在的请求完成，则堵塞状态会解除，继续向下执行，得到正确结果
		return c.val, c.err
	}
	//如果不存在对该key的请求，则本线程要进行实际的请求，保持m的锁定状态
	//创建一个实际请求结构体
	c := new(call)
	//为了保证其他的相同请求的堵塞
	c.wg.Add(1)
	//组织好映射关系
	g.m[key] = c
	//解锁m
	g.mu.Unlock()
	
	//执行真正的请求函数，得到对该key请求的结果
	c.val, c.err = fn()
	//得到结果后取消其他请求的堵塞
	c.wg.Done()

	//该次请求完成后，要从已存在请求map中删掉
	g.mu.Lock()
	delete(g.m, key)
	g.mu.Unlock()
	
	//返回请求结果
	return c.val, c.err
}
```
#### 使用实例
```go
package main

import (
	"fmt"
	"sync"
	"time"

	"github.com/golang/groupcache/singleflight"
)

func NewDelayReturn(dur time.Duration, n int) func() (interface{}, error) {
	return func() (interface{}, error) {
		time.Sleep(dur)
		return n, nil
	}
}

func main() {
	g := singleflight.Group{}
	wg := sync.WaitGroup{}
	wg.Add(2)
	go func() {
		ret, err := g.Do("key", NewDelayReturn(time.Second*1, 1))
		if err != nil {
			panic(err)
		}
		fmt.Printf("key-1 get %v\n", ret)
		wg.Done()
	}()
	go func() {
		time.Sleep(100 * time.Millisecond) // make sure this is call is later
		ret, err := g.Do("key", NewDelayReturn(time.Second*2, 2))
		if err != nil {
			panic(err)
		}
		fmt.Printf("key-2 get %v\n", ret)
		wg.Done()
	}()
	wg.Wait()
}
```
执行结果(耗时： 1.019s)

```
key-2 get 1
key-1 get 1
```
#### 参考资料：
<https://my.oschina.net/goskyblue/blog/656413>

<http://www.voidcn.com/article/p-kjpxomam-baw.html>
