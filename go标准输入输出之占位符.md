## go标准输入输出之占位符

#### 简介

fmt 包实现了格式化 I/O 函数，类似于 C 的 printf 和 scanf。格式“占位符”衍生自 C，但比 C 更简单。
#### 打印
占位符：
##### [一般]

　　%v	相应值的默认格式。在打印结构体时，“加号”标记（%+v）会添加字段名

　　%#v	相应值的 Go 语法表示

　　%T	相应值的类型的 Go 语法表示

　　%%	字面上的百分号，并非值的占位符

##### [布尔]

　　%t	单词 true 或 false。

##### [整数]

　　%b	二进制表示

　　%c	相应 Unicode 码点所表示的字符

　　%d	十进制表示

　　%o	八进制表示

　　%q	单引号围绕的字符字面值，由 Go 语法安全地转义

　　%x	十六进制表示，字母形式为小写 a-f

　　%X	十六进制表示，字母形式为大写 A-F

　　%U	Unicode 格式：U+1234，等同于 "U+%04X"

##### [浮点数及其复合构成]

　　%b	无小数部分的，指数为二的幂的科学计数法，与 strconv.FormatFloat 的 'b' 转换格式一致。例如 -123456p-78

　　%e	科学计数法，例如 -1234.456e+78

　　%E	科学计数法，例如 -1234.456E+78

　　%f	有小数点而无指数，例如 123.456

　　%g	根据情况选择 %e 或 %f 以产生更紧凑的（无末尾的 0）输出

　　%G	根据情况选择 %E 或 %f 以产生更紧凑的（无末尾的 0）输出

##### [字符串与字节切片]

　　%s	字符串或切片的无解译字节

　　%q	双引号围绕的字符串，由 Go 语法安全地转义

　　%x	十六进制，小写字母，每字节两个字符

　　%X	十六进制，大写字母，每字节两个字符

##### [指针]

　　%p	十六进制表示，前缀 0x

##### [注意]

　　这里没有 'u' 标记。若整数为无符号类型，他们就会被打印成无符号的。类似地， 这里也不需要指定操作数的大小（int8，int64）。

　　宽度与精度的控制格式以 Unicode 码点为单位。（这点与 C 的 printf 不同， 它以字节数为单位。）二者或其中之一均可用字符 '*' 表示， 此时它们的值会从下一个操作数中获取，该操作数的类型必须为 int。
　　
##### 示例代码：
```go
//格式化占位符
package main

import (
	"fmt"
)

type Human struct {
	Name string
}

func main() {
	people := Human{
		Name:"zhangsan",
	}

	//普通占位符
	fmt.Printf("%v\n", people)//{zhangsan}
	fmt.Printf("%+v\n", people)//{Name:zhangsan}
	fmt.Printf("%#v\n", people)//main.Human{Name:"zhangsan"}
	fmt.Printf("%T\n", people)//main.Human
	fmt.Printf("%%\n")//%

	//布尔占位符
	fmt.Printf("%t\n", true)//true

	//整型占位符
	fmt.Printf("%b\n", 12)//1100
	fmt.Printf("%c\n", 0x4E2D)//中
	fmt.Printf("%d\n", 12)//12
	fmt.Printf("%o\n", 12)//14
	fmt.Printf("%q\n", 0x4E2D)//'中'
	fmt.Printf("%x\n", 12)//c
	fmt.Printf("%X\n", 12)//C
	fmt.Printf("%U\n", 0x4E2D)//U+4E2D

	//浮点型或复数占位符
	fmt.Printf("%b\n", 1234.5678)//5429686605511341p-42
	fmt.Printf("%e\n", 1234.5678)//1.234568e+03
	fmt.Printf("%E\n", 1234.5678)//1.234568E+03
	fmt.Printf("%f\n", 1234.5678)//1234.567800
	fmt.Printf("%g\n", 1234.5678)//1234.5678
	fmt.Printf("%G\n", 1234.5678)//1234.5678

	//字符串与字节切片
	fmt.Printf("%s\n", []byte("go"))//go
	fmt.Printf("%q\n", "go")//"go"
	fmt.Printf("%x\n", "go")//676f
	fmt.Printf("%X\n", "go")//676F

	//指针占位符
	fmt.Printf("%p\n", &people)//0xc00007e030
}
```

#### 参考资料
[https://golang.org/pkg/fmt/](https://golang.org/pkg/fmt/)

[https://studygolang.com/articles/2644](https://studygolang.com/articles/2644)