## golang并发--syn/atomic包
sync/atomic包提供了原子操作，即进行过程中不能被中断的操作。

该包提供的可进行原子操作类型包括int32,int64,uint32,uint64,uintptr,unsafe.Pointer，共六个。

这些函数提供的原子操作共有五种：增减，比较并交换，载入，存储和交换。

### 增减 Add
函数名称都以Add为前缀，并后跟针对的具体类型的名称：
>
func AddInt32(addr *int32, delta int32) (new int32)
>
func AddInt64(addr *int64, delta int64) (new int64)
>
func AddUint32(addr *uint32, delta uint32) (new uint32)
>
func AddUint64(addr *uint64, delta uint64) (new uint64)
>
func AddUintptr(addr *uintptr, delta uintptr) (new uintptr)

被操作的类型只能是数值类型，int32,int64,uint32,uint64,uintptr类型可以使用原子增或减操作。

第一个参数值必须是一个指针类型的值，以便施加特殊的CPU指令。

第二个参数值的类型和第一个被操作值的类型总是相同的。

示例:

```go
package main

import (
	"sync/atomic"
	"fmt"
)

func click(count *int, ch chan bool) {
	for i := 0; i < 10000; i++ {
		*count += 1
	}
	ch <- true
}

func clickat(count *int32, ch chan bool) {
	for i := 0; i < 10000; i++ {
		atomic.AddInt32(count, 1)
	}
	ch <- true
}

func main() {
	ch := make(chan bool, 10)
	count1 := 0
	count2 := int32(0)

	for i := 0; i < 5; i++ {
		go click(&count1, ch)
	}
	for i := 0; i < 5; i++ {
		go clickat(&count2, ch)
	}

	for i := 0; i < 10; i++ {
		<- ch
	}

	fmt.Println("count1: ", count1)
	fmt.Println("count2: ", count2)
}
```

### 比较并交换CAS
Compare And Swap 简称CAS，在sync/atomic包种，这类原子操作由名称以‘CompareAndSwap’为前缀的若干个函数代表。

声明如下:
>
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)
>
func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)
>
func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool)
>
func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)
>
func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)
>
func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)

调用函数后，会先判断参数addr指向的被操作值与参数old的值是否相等，仅当此判断得到肯定的结果之后，才会用参数new代表的新值替换掉原先的旧值，否则操作就会被忽略。

所以一般需要用for循环不断进行尝试,直到成功为止。

示例:

```go
package main

import (
	"fmt"
	"time"
	"sync/atomic"
)

func c32To42(count *int32, index int) {
	if atomic.CompareAndSwapInt32(count, 32, 42) {
		fmt.Println("32 to 42 successful, id: ", index)
	}else {
		fmt.Println("32 to 42 faild, id: ", index)
	}
}

func c42To32(count *int32, index int) {
	if atomic.CompareAndSwapInt32(count, 42, 32) {
		fmt.Println("42 to 32 successful, id: ", index)
	}else {
		fmt.Println("42 to 32 faild, id: ", index)
	}
}

func main() {
	count := int32(32)

	for i := 0; i < 50; i++ {
		go c32To42(&count, i)
		go c42To32(&count, i)
	}

	time.Sleep(time.Second * 5)
}

```

### 载入Load
上面的比较并交换案例中用 v := value为变量v赋值，但要注意，在进行读取value的操作的过程中,其他对此值的读写操作是可以被同时进行的,那么这个读操作很可能会读取到一个只被修改了一半的数据.

所以 , 我们要使用sync/atomic代码包同样为我们提供了一系列的函数，以Load为前缀(载入)，来确保这样的糟糕事情不会发生。

这相当于原子读。

atomic.LoadInt32接受一个*int32类型的指针值，返回该指针指向的那个值：
>
func LoadInt32(addr *int32) (val int32)
>
func LoadInt64(addr *int64) (val int64)
>
func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)
>
func LoadUint32(addr *uint32) (val uint32)
>
func LoadUint64(addr *uint64) (val uint64)
>
func LoadUintptr(addr *uintptr) (val uintptr)

示例：

```go
package main

import (
	"fmt"
	"sync/atomic"
	"time"
)

func addInt(count *int32) {
	for i := 0; i < 100; i++ {
		time.Sleep(time.Nanosecond)
		atomic.AddInt32(count, 1)
		if i == 50 {
			fmt.Println("count1: ", atomic.LoadInt32(count), *count)
		}	
	}
}

func main() {
	count1 := int32(0)

	for i := 0; i < 5; i++ {
		go addInt(&count1)
	}

	time.Sleep(time.Second * 3)
}
```
### 存储Store
与读取操作相对应的是写入操作。 而sync/atomic包也提供了与原子的载入函数相对应的原子的值存储函数， 以Store为前缀。

在原子地存储某个值的过程中，任何CPU都不会进行针对同一个值的读或写操作，原子的值存储操作总会成功，因为它并不会关心被操作值的旧值是什么。
>
func StoreInt32(addr *int32, val int32)
>
func StoreInt64(addr *int64, val int64)
>
func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)
>
func StoreUint32(addr *uint32, val uint32)
>
func StoreUint64(addr *uint64, val uint64)
>
func StoreUintptr(addr *uintptr, val uintptr)

示例：

```go
package main

import "fmt"
import "time"
import "sync/atomic"

func read(num *int32) {
	for {
    	if *num==32 {
        	fmt.Println("ops:", 32)
    	}
    	if *num==42 {
        	fmt.Println("ops:", 42)
    	}
    	time.Sleep(time.Nanosecond)
	}
}

func main() {

	var ops int32 = 0

	for i := 0; i < 4; i++ {
    	go read(&ops)
	}

	atomic.StoreInt32(&ops, 32)

	time.Sleep(time.Nanosecond*5)

	ops = 42					//42的输出会不稳定。
	fmt.Println("changed")

	time.Sleep(time.Nanosecond*20)
}
```
### 交换Swap
与CAS操作不同，原子交换操作不会关心被操作的旧值。
它会直接设置新值。
它会返回被操作值的旧值。
此类操作比CAS操作的约束更少，同时又比原子载入操作的功能更强。
>
func SwapInt32(addr *int32, new int32) (old int32)
>
func SwapInt64(addr *int64, new int64) (old int64)
>
func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)>
>
func SwapUint32(addr *uint32, new uint32) (old uint32)
>
func SwapUint64(addr *uint64, new uint64) (old uint64)
>
func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)

示例：

```go
package main

import (
	"fmt"
	"time"
	"sync/atomic"
)

func c42To32(count *int32, index int) {
	fmt.Println("id: ", index, " old: ", atomic.SwapInt32(count, 32), " new: ", *count)
}

func main() {
	count := int32(42)

	for i := 0; i < 50; i++ {
		go c42To32(&count, i)
	}

	time.Sleep(time.Second * 5)
}

```
### 参考资料：
[https://golang.org/pkg/sync/atomic/](https://golang.org/pkg/sync/atomic/)

[https://github.com/astaxie/gopkg/tree/master/sync/atomic](https://github.com/astaxie/gopkg/tree/master/sync/atomic)

[https://www.kancloud.cn/digest/batu-go/153537](https://www.kancloud.cn/digest/batu-go/153537)