##Go 并发常用知识点实例

Go中天然的支持并发，Go允许使用go语句开启一个新的运行期线程，即 goroutine，以一个不同的、新创建的goroutine来执行一个函数。同一个程序中的所有goroutine共享同一个地址空间。

Goroutine非常轻量，除了为之分配的栈空间，其所占用的内存空间微乎其微。并且其栈空间在开始时非常小，之后随着堆存储空间的按需分配或释放而变化。内部实现上，goroutine会在多个操作系统线程上多路复用。如果一个goroutine阻塞了一个操作系统线程，例如：等待输入，这个线程上的其他goroutine就会迁移到其他线程，这样能继续运行。开发者并不需要关心/担心这些细节。

下面整理了并发编程中可能会经常用到的基本知识点，主要是channel的使用以及sync包的使用。

1、channel使用

channel通常用来在多个线程之间传递消息或者进行同步，channel的基本语法这里不再赘述，直接上一个实例代码：

```go
package main

import (
	"fmt"
	"time"
)

func worker(dead <-chan bool, index int) {
	fmt.Println("Worker ", index, " Start!")
	for {
		select {
			case <-dead:
				fmt.Println("Worker ", index, " Done!")
				break
			default :
				//fmt.Println("Worker ", index, "Doing...") 
		}
	}
}

func worker2(done chan<- int, index int) {
	fmt.Println("Worker ", index, " Start!")
	time.Sleep(time.Second * 5)
	done <- index
}

func main() {
    
    /*
	//单个channel，unbuffered channel 同步用法
	c := make(chan bool)
	go func(){
			fmt.Println("goruntine child Test!")
			c <- true
		}()
	<- c
	fmt.Println("goruntine main Test!")	
    */

    
    /*
    //单个channel，buffered channel 异步用法
    c := make(chan int, 3)
    go func(){
    		for i := 0; i < 4; i++ {
    			c <- i
    			fmt.Println("write to c: ", i)
    		}
    	}()
	for j := 0; j < 4; j++ {
		fmt.Println("read from c: ", <-c)
	}
	*/

	
	/*
	//协同多个goruntine，unbuffeered channel 同步用法
	c := make(chan bool)
	fmt.Println("Start Master process!")
	for i := 1; i < 5; i++ {
		go func(index int){
			fmt.Println("Worker ", index, " Start!")
			<- c
			fmt.Println("Worker ", index, " Done!")
		}(i)
	}
	for j := 1; j < 5; j++ {
		c <- true
	}
	fmt.Println("Master Done!")
	*/
	
	
	/*
	//select 用法
	die := make(chan bool)
	for i := 0; i < 5; i++ {
		go worker(die, i)
	}

	time.Sleep(time.Second)
	for i := 0; i < 5; i++ {
		die <- true
	}
	*/
	
	
	fmt.Println("Master Start!")
	c1 := make(chan int)
	c2 := make(chan int)
	c3 := make(chan int)
	c4 := make(chan int)
	go worker2(c1, 1)
	go worker2(c2, 2)
	go worker2(c3, 3)
	go worker2(c4, 4)
	num := 0
Done:
	for {
		select {
		case <- c1 :
			fmt.Println("Worker 1 Done!")
			num += 1
		case <- c2 :
			fmt.Println("Worker 2 Done!")
			num += 1
		case <- c3 :
			fmt.Println("Worker 3 Done!")
			num += 1
		case <- c4 :
			fmt.Println("Worker 4 Done!")
			num += 1
		default :
			if num >= 4 {
				break Done
			}
		}
	}
	fmt.Println("Master Done!")
}
```
2、sync.Cond 使用

Cond是条件等待，条件等待通过 Wait 让例程等待，通过 Signal 让一个等待的例程继续，通过 Broadcast 让所有等待的例程继续。

在 Wait 之前应当手动为 c.L 上锁，Wait 结束后手动解锁。为避免虚假唤醒，需要将 Wait 放到一个条件判断循环中。官方要求的写法如下：

```go
c.L.Lock()
for !condition() {
    c.Wait()
}
// 执行条件满足之后的动作...
c.L.Unlock()
```
Cond 在开始使用之后，不能再被复制。

示例代码如下：

```go
package main

import (
	"fmt"
	"time"
	"sync"
)

func waiter(cond *sync.Cond, id int){
	cond.L.Lock()
	cond.Wait()
	cond.L.Unlock()

	fmt.Println("Waiter ", id, "wake up!")
}

func main() {
	locker := new(sync.Mutex)
	cond := sync.NewCond(locker)

	for i := 0; i < 3; i++ {
		go waiter(cond, i)
	}
	time.Sleep(time.Second * 3)

	cond.L.Lock()
	cond.Signal()
	cond.L.Unlock()

	for i := 3; i < 5; i++ {
		go waiter(cond, i)
	}
	time.Sleep(time.Second * 3)

	cond.L.Lock()
	cond.Signal()
	cond.L.Unlock()

	cond.L.Lock()
	cond.Broadcast()
	cond.L.Unlock()

	time.Sleep(time.Second * 5)
}
```
3、sync.Mutex使用

Mutex是互斥锁，用来保证在任一时刻，只能有一个例程访问某对象。Mutex 的初始值为解锁状态。Mutex 通常作为其它结构体的匿名字段使用，使该结构体具有 Lock 和 Unlock 方法。

Mutex 可以安全的在多个例程中并行使用。

示例代码如下：

```go
package main

import (
	"fmt"
	"sync"
)

func click(c chan bool, count *int) {
	for i := 0; i < 1000; i++ {
		*count += 1
	}
	c <- true
}

func clickWithMutex(c chan bool, count *int, m *sync.Mutex) {
	for i := 0; i < 1000; i++ {
		m.Lock()
		*count += 1
		m.Unlock()
	}
	c <- true
}

func main() {
	m := new(sync.Mutex)
	count1, count2 := 0, 0
	c := make(chan bool, 10)

	for i := 0; i < 5; i++ {
		go click(c, &count1)
	}

	for i := 0; i < 5; i++ {
		go clickWithMutex(c, &count2, m)
	}

	for i := 0; i < 10; i++ {
		<- c
	}

	fmt.Println("count1: ", count1)
	fmt.Println("count2: ", count2)
}
```
4、sync.Once使用

Once是单次执行。

Once 的作用是多次调用但只执行一次，Once 只有一个方法，Once.Do()，向 Do 传入一个函数，这个函数在第一次执行 Once.Do() 的时候会被调用，以后再执行 Once.Do() 将没有任何动作，即使传入了其它的函数，也不会被执行，如果要执行其它函数，需要重新创建一个 Once 对象。

Once 可以安全的在多个例程中并行使用。

示例代码：

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	once := new(sync.Once)
	ch := make(chan bool, 5)

	for i := 0; i < 5; i++ {
		go func(x int){
			once.Do(func(){
				fmt.Println("Worker Do: ", x)
			})
			fmt.Println("Master Do: ", i)
			ch <- true
		}(i)
	}

	for j := 0; j < 5; j++ {
		<- ch
	}

}
```

5、sync.RWMutex使用

RWMutex是读写互斥锁。

RWMutex 比 Mutex 多了一个“读锁定”和“读解锁”，可以让多个例程同时读取某对象。RWMutex 的初始值为解锁状态。RWMutex 通常作为其它结构体的匿名字段使用。

Mutex 可以安全的在多个例程中并行使用。

示例代码：

```go
package main

import (
	"fmt"
	"sync"
)

func clickWithRWMutex(m *sync.RWMutex, total *int, ch chan int) {
	for i := 0; i < 1000; i++ {
		m.Lock()
		*total += 1
		m.Unlock()

		if i == 500 {
			m.RLock()
			fmt.Println("Middle Num: ", *total)
			m.RUnlock()
		}
	}
	ch <- 1
}

func main() {
	m := new(sync.RWMutex)
	ch := make(chan int, 10)
	count := 0

	for i := 0; i < 10; i++ {
		go clickWithRWMutex(m, &count, ch)
	}

	for i := 0; i < 10; i++ {
		<- ch
	}

	fmt.Println("Count: ", count)
}
```

6、sync.WaitGroup使用

WaitGroup是组等待。

WaitGroup 用于等待一组例程的结束。主例程在创建每个子例程的时候先调用 Add 增加等待计数，每个子例程在结束时调用 Done 减少例程计数。之后，主例程通过 Wait 方法开始等待，直到计数器归零才继续执行。

示例代码：

```go
package main

import (
	"sync"
	"fmt"
)

func wgProcess(wg *sync.WaitGroup, index int) {
	fmt.Println("WgProcess ", index, "is going!")
	wg.Done()
}

func main() {
	wg := new(sync.WaitGroup)

	for i := 0; i < 20; i++ {
		wg.Add(1)
		go wgProcess(wg, i)
	}

	wg.Wait()
	fmt.Println("All is Done!")
}
```

7、sync/atomic 使用

sync/atomic是原子操作，具体未完待续。。。。。。


##### 参考资料：

[http://blog.xiayf.cn/2015/05/20/fundamentals-of-concurrent-programming/](http://blog.xiayf.cn/2015/05/20/fundamentals-of-concurrent-programming/)

[https://github.com/astaxie/gopkg/tree/master/sync](https://github.com/astaxie/gopkg/tree/master/sync)

[https://www.cnblogs.com/golove/p/5918082.html](https://www.cnblogs.com/golove/p/5918082.html)