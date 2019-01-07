## Go fmt 包实例

fmt包提供格式化输入输出，主要有下面几个函数：
>
func Printf(format string, a ...interface{}) (n int, err error)
>
func Print(a ...interface{}) (n int, err error)
>
func Println(a ...interface{}) (n int, err error)
>
func Sprintf(format string, a ...interface{}) string
>
func Sprint(a ...interface{}) string
>
func Sprintln(a ...interface{}) string
>
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
>
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
>
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
>
func Scanf(format string, a ...interface{}) (n int, err error)
>
func Scan(a ...interface{}) (n int, err error)
>
func Scanln(a ...interface{}) (n int, err error)
>
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
>
func Sscan(str string, a ...interface{}) (n int, err error)
>
func Sscanln(str string, a ...interface{}) (n int, err error)
>
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
>
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
>
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
>
func Errorf(format string, a ...interface{}) error

函数实例：

```go
//fmt格式化输入输出函数
package main

import (
	"fmt"
	"os"
)

func main() {
	
	//Print 系列
	fmt.Printf("Printf: %s\n", "按格式打印，空格换行需自行添加")
	fmt.Print("Print: ", "默认格式打印，非字符串自动加空格，换行需自行添加\n")
	fmt.Println("Println:", "默认格式打印，所有元素自动加空格，自动添加换行")

	//Sprint系列
	str := fmt.Sprintf("Sprintf: %s\n", "按格式打印，空格换行需自行添加")
	fmt.Print(str)
	str = fmt.Sprint("Sprint: ", "默认格式打印，非字符串自动加空格，换行需自行添加\n")
	fmt.Print(str)
	str = fmt.Sprintln("Sprintln:", "默认格式打印，所有元素自动加空格，自动添加换行")
	fmt.Print(str)

	//Fprint系列
	fmt.Fprintf(os.Stdout, "Fprintf: %s\n", "按格式打印，空格换行需自行添加")
	fmt.Fprint(os.Stdout, "Fprint: ", "默认格式打印，非字符串自动加空格，换行需自行添加\n")
	fmt.Fprintln(os.Stdout, "Fprintln:", "默认格式打印，所有元素自动加空格，自动添加换行")

	
	//Scanf系列
	var a, b, c int
	
	fmt.Scanf("%d%d%d", &a, &b, &c)//输入时用任意空格或tab隔开，不能换行
	fmt.Println(a, b, c)
	fmt.Scanf("%d,%d,%d", &a, &b, &c)//输入时要用逗号隔开，要跟引号内保持一致,其他字符均可
	fmt.Println(a, b, c)
	fmt.Scan(&a, &b, &c)//输入时用任意空格、TAB或者回车隔开，直到达到参数数量为止
	fmt.Println(a, b, c)
	fmt.Scanln(&a, &b, &c)//输入时用任意空格、TAB隔开，不能用换行，换行即终止
	fmt.Println(a, b, c)

	input := "12\n\n34\n56"
	fmt.Sscanf(input, "%d\n\n%d\n%d", &a, &b, &c)//input与%d串保持一致，若%d之间无任何字符，则input之间可为任意空格或TAB
	fmt.Println(a, b, c)
	fmt.Sscan(input, &a, &b, &c)//input中可用任意空格、tab、回车间隔，直到到达参数数量为止
	fmt.Println(a, b, c)
	input2 := "23 45 67"
	fmt.Sscanln(input2, &a, &b, &c)//input中只能用任意空格、tab间隔，不能换行，换行即终止
	fmt.Println(a, b, c)
	

	fmt.Fscanf(os.Stdin, "%d,%d,%d", &a, &b, &c)//输入时要用逗号隔开，要跟引号内保持一致,其他字符均可。无间隔时默认输入时用任意空格或tab隔开，不能换行
	fmt.Println(a, b, c)
	fmt.Fscan(os.Stdin, &a, &b, &c)//输入时用任意空格、TAB或者回车隔开，直到达到参数数量为止
	fmt.Println(a, b, c)
	fmt.Fscanln(os.Stdin, &a, &b, &c)//输入时用任意空格、TAB隔开，不能用换行，换行即终止
	fmt.Println(a, b, c)

	//Errorf系列
	err := fmt.Errorf("Error: %s", "Test error!")//生成一个error类型
	fmt.Println(err)

}
```

#### 参考资料：
[https://github.com/astaxie/gopkg/tree/master/fmt](https://github.com/astaxie/gopkg/tree/master/fmt)

[https://golang.org/pkg/fmt/](https://golang.org/pkg/fmt/)