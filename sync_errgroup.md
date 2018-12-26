## Go sync/errgroup 实例

golang中增加了一个errgroup包，它在sync.WaitGroup功能的基础上，增加了错误传递，以及在发生不可恢复的错误时取消整个goroutine集合，或者等待超时。

其中包含的函数如下：

>func WithContext(ctx context.Context) (*Group, context.Context)

>func (g *Group) Go(f func() error)

>func (g *Group) Wait() error

下面先上一个实例，这个例子比较简单，只用到了Go以及Wait两个函数。

```
package main

import (
	"fmt"
	"golang.org/x/sync/errgroup"
	"errors"
)

func output(num int) (int, error) {
	if num < 0 {
		return 0, errors.New("math: square root error!")
	}
	return num, nil
}

func main() {
	group := new(errgroup.Group)
	nums := []int{
		-1,
		0,
		1,
	}

	for _, num := range nums {
		num := num
		group.Go(func() error {
			res, err := output(num)
			fmt.Println(res)
			return err
		})
	}

	if err := group.Wait(); err != nil {
		fmt.Println("Get errors: ", err)
	}else {
		fmt.Println("Get all num successfully!")
	}

}
```

再下面这个例子使用了WithContext函数,这主要是便于在各个goruntine之间传递数据。

```
package main

import (
	"fmt"
	"time"

	"golang.org/x/sync/errgroup"
	"golang.org/x/net/context"
)

func checkGoroutineErr(errCtx context.Context) error {
	select {
	case <-errCtx.Done():
		return errCtx.Err()
	default:
		return nil
	}
}

func main() {
	ctx , cancel := context.WithCancel(context.Background())
	group, errCtx := errgroup.WithContext(ctx)

	for i := 0; i < 3; i++ {
		index := i
		group.Go(func() error {
			fmt.Println("index=", index)
			if index == 0 {
				fmt.Println("index == 0, end!")
			}else if index == 1 {
				fmt.Println("index == 1, start...")
				cancel()
				fmt.Println("inde == 1, has error!")
			}else if index == 2 {
				fmt.Println("index == 2, start...")
				time.Sleep(time.Second * 3)
				if err := checkGoroutineErr(errCtx); err != nil {
					return err
				}
				fmt.Println("index == 2, has done!")
			}
			return nil
		})
	}

	err := group.Wait()
	if err != nil {
		fmt.Println("Get error: ", err)
	}else {
		fmt.Println("All Done!")
	}
}
```
### 参考资料
[https://blog.csdn.net/jiankunking/article/details/78818953](https://blog.csdn.net/jiankunking/article/details/78818953)
[https://godoc.org/golang.org/x/sync/errgroup](https://godoc.org/golang.org/x/sync/errgroup)
