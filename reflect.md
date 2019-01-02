## Go reflect 应用场景实例
reflect包实现了反射机制。

首先，reflect包最核心的两个数据类型我们必须知道，一个是Type，一个是Value。

Type就是定义的类型的一个数据类型，Value是值的类型。

反射就是用来检测存储在接口变量内部(值value；类型concrete type) pair对的一种机制。那么在Golang的reflect反射包中有什么样的方式可以让我们直接获取到变量内部的信息呢？ 它提供了两种类型（或者说两个方法）让我们可以很容易的访问接口变量内容，分别是reflect.ValueOf() 和 reflect.TypeOf()。

#### TypeOf(i interface{}) Type
其中常用的的内置方法以及一些函数如下：

```
func ChanOf(dir ChanDir, t Type) Type    // 创建反射的信道。其实就是类似这样 reflect.TypeOf(chan int)
func MapOf(key, elem Type) Type    // 创建反射的Map。其实就是类似这样 reflect.TypeOf(map[int]string)
func SliceOf(t Type) Type    // 创建反射的Slice。其实就是类似这样 reflect.TypeOf([]int)
func PtrTo(t Type) Type    // 返回元素 t 的指针类型
func TypeOf(i interface{}) Type    // 返回反射interface{}接口的类型
NumMethod() int    // 函数总数量，在struct结构中
Method(int) Method    // 指定返回函数的 Method 类型，在struct结构中
MethodByName(string) (Method, bool)    // 使用“字符串”函数名称返回函数的 Method 类型，在struct结构中
NumField() int    // 字段总数量，在struct结构中
Field(i int) StructField    // 指定返回字段的 StructField 类型，在struct结构中
FieldByIndex(index []int) StructField    // 指定返回“嵌套”字段的 StructField 类型，在struct结构中
FieldByName(name string) (StructField, bool)    // 使用“字符串”字段名称返回字段的 StructField 类型，在struct结构中
FieldByNameFunc(match func(string) bool) (StructField, bool)    // 传入字段“字符串”名称，并判断，func 返回 true，返回字段的 StructField 类型，在struct结构中
NumIn() int    // 函数输入参数总数量
In(i int) Type    // 返回函数输入参数的第i个类型 Type
NumOut() int    // 函数输出参数总数量
Out(i int) Type    // 返回函数输出参数的第i个类型 Type
Align() int    // 在分配在内存时的此类型的一个值（以字节为单位）的对齐。
FieldAlign() int    // 返回字段对齐的值（以字节为单位）
Name() string    // 变量名称或字段的名称
PkgPath() string    // 变量的（包）路径名
Size() uintptr    // 值的数据大小（以字节为单位）
String() string    // （包）路径名称+类型名称
Kind() Kind    // 变量的类型
Implements(u Type) bool    // 判断是否存在与 u 相同的接口
AssignableTo(u Type) bool    // 判断值是否可分配给 u
ConvertibleTo(u Type) bool    // 判断值是否可以转换为 u 类型
Bits() int    // 返回类型比特的大小
ChanDir() ChanDir    // 返回信道的方向
IsVariadic() bool    // 返回函数的类型最后一个输入参数是否是“...”参数。
Elem() Type    // 指针指向内存地址
Key() Type    // 返回 Map 键Key的类型
Len() int    // 返回 Array 的长度
```
#### ValueOf(i interface{}) Value
其中一些常用的函数如下：

```
func Append(s Value, x ...Value) Value    // 追加Slice
func AppendSlice(s, t Value) Value    // 批量追加Slice
func Indirect(v Value) Value    // 返回指针源内存地址
func MakeChan(typ Type, buffer int) Value    // 初始化信道
func MakeFunc(typ Type, fn func(args []Value) (results []Value)) Value    // 初始化函数，并可以对函数的参数进得修改操作。
func MakeMap(typ Type) Value    // 初始化Map
func MakeSlice(typ Type, len, cap int) Value    // 初始化Slice
func New(typ Type) Value    // 初始化并返回指针
func NewAt(typ Type, p unsafe.Pointer) Value    // 初始化 p 并返回指针，转向给 typ
func Zero(typ Type) Value    // 初始化 typ 为零值
func ValueOf(i interface{}) Value    // 返回反射interface{}接口的值
func (v Value) Elem() Value    // 指针指向内存地址
func (v Value) Type() Type    // 返回类型 rflect.Type
func (v Value) Convert(t Type) Value    // 转换v 为 t 同一种类型
func (v Value) NumField() int    // 字段总数量，在struct结构中
func (v Value) Field(i int) Value    // 指定返回字段的 Value 类型，在struct结构中
func (v Value) FieldByIndex(index []int) Value    // 指定返回“嵌套”字段的 Value 类型，在struct结构中
func (v Value) FieldByName(name string) Value    // 使用“字符串”字段名称返回字段的 Value 类型，在struct结构中
func (v Value) FieldByNameFunc(match func(string) bool) Value    // 传入字段“字符串”名称，并判断，func 返回 true，返回字段的 Value 类型，在struct结构中
func (v Value) NumMethod() int    // 函数总数量，在struct结构中
func (v Value) Method(i int) Value    // 指定返回函数的 Value 类型，在struct结构中
func (v Value) MethodByName(name string) Value    // 使用“字符串”函数名称返回函数的 Value 类型，在struct结构中
func (v Value) Index(i int) Value    // 返回Array或Slice类型的第i个切片，在struct结构中
func (v Value) Kind() Kind    // 值的类型
func (v Value) Call(in []Value) []Value    // 调用函数，in 切片装入参数，传入函数
func (v Value) CallSlice(in []Value) []Value    // 调用函数，in 切片装入参数，传入函数。用于可变参数函数
func (v Value) IsValid() bool    // 判断值是否是零值
func (v Value) IsNil() bool    // 判断值是否是 nil，限制支持 Chan，Func，Interface，Map，Ptr，或Slice
func (v Value) CanInterface() bool    // 判断值是否可以做为 interface{} 类型读出
func (v Value) Interface() (i interface{})    // 以接口类型读出数据
func (v Value) InterfaceData() [2]uintptr    // 返回一对作为uintptr的接口值
func (v Value) Slice(beg, end int) Value    // 返回指定长度的切片
func (v Value) CanAddr() bool    // 判断是否是可以寻址
func (v Value) Addr() Value    // 返回指针值的地址
func (v Value) UnsafeAddr() uintptr    // 返回安全指针指向v的数据
func (v Value) CanSet() bool    // 判断是否可以写入值
func (v Value) Set(x Value)    // 写入新值，支持所有类型
func (v Value) Bool() bool    // 返回 Bool 类型的值
func (v Value) SetBool(x bool)    // 写入 Bool 类型的值
func (v Value) Bytes() []byte    // 返回 Byte 类型的值
func (v Value) SetBytes(x []byte)    // 写入 Byte 类型的值
func (v Value) Int() int64    // 返回 Int 类型的值
func (v Value) OverflowInt(x int64) bool    // 判断 Int 类型的值承受范围
func (v Value) SetInt(x int64)    // 写入 Int 类型的值
func (v Value) Uint() uint64    // 返回 Uint 类型的值
func (v Value) OverflowUint(x uint64) bool    // 判断 Uint 类型的值承受范围
func (v Value) SetUint(x uint64)    // 写入 Uint 类型的值
func (v Value) Float() float64    // 返回 Float 类型的值
func (v Value) OverflowFloat(x float64) bool    // 判断 Float 类型的值承受范围
func (v Value) SetFloat(x float64)    // 写入 Float 类型的值
func (v Value) Complex() complex128    // 返回 Complex 类型的值
func (v Value) OverflowComplex(x complex128) bool    // 判断 Complex 类型的值承受范围
func (v Value) SetComplex(x complex128)    // 写入 Complex 类型的值
func (v Value) Pointer() uintptr    // 返回指针（整数）
func (v Value) SetPointer(x unsafe.Pointer)    // 写入新的指针
func (v Value) String() string    // 返回 String 类型的值
func (v Value) SetString(x string)    // 写入 String 类型的值
func (v Value) MapKeys() []Value    // 返回 Map 中的所有 Key 名称
func (v Value) MapIndex(key Value) Value    // 返回 Map 中 Key 的值
func (v Value) SetMapIndex(key, val Value)    // 设置 Mep 的 值
func (v Value) Len() int    // 返回 Slice，Array，Chan，Map，String 长度
func (v Value) Cap() int    // 返回 Slice，Array，Chan 容量
func (v Value) SetLen(n int)    // 改变 Slice 长度
func (v Value) Recv() (x Value, ok bool)    // 信道接收
func (v Value) Send(x Value)    // 信道发送
func (v Value) TryRecv() (x Value, ok bool)    // 信道尝式接收
func (v Value) TrySend(x Value) bool    // 信道尝式发送
func (v Value) Close()    // 关闭信道
```

reflect的一些典型的应用场景如下：
##### 1、动态调用函数

```
package main

import (
	"fmt"
	"reflect"
	"errors"
)

//动态调用函数(有参，无参，有返回值)
type T struct {}

func (t *T) Do() {
	fmt.Println("Hello, world!")
}

func (t *T) DoWithPara(name string, index int) {
	fmt.Println("Hello, ", name, index)
}

func (t *T) DoWithErr() (string, error){
	return "hello, world", errors.New("new error")
}

func main() {
	name1 := "Do"
	name2 := "DoWithPara"
	name3 := "DoWithErr"
	t := new(T)

	reflect.ValueOf(t).MethodByName(name1).Call(nil)

	name := reflect.ValueOf("world")
	index := reflect.ValueOf(1234)
	in := []reflect.Value{name, index}
	reflect.ValueOf(t).MethodByName(name2).Call(in)

	ret := reflect.ValueOf(t).MethodByName(name3).Call(nil)
	fmt.Println(ret[0], ret[1].Interface().(error))
}

```
##### 2、struct tag 解析

```
package main

import (
	"fmt"
	"reflect"
)

//struct tag 解析
type T struct {
	A int `json:"aaa" test:"aaatest"`
	B string `json:"bbb" test:"bbbtest"`
}

func main() {
	t := T{
		A: 123456,
		B: "helloworld",
	}
	tt := reflect.TypeOf(t)
	for i := 0; i < tt.NumField(); i++ {
		field := tt.Field(i)
		if json, ok := field.Tag.Lookup("json"); ok {
			fmt.Println(json)
		}

		test := field.Tag.Get("test")
		fmt.Println(test)
	}
}
```
##### 3、类型转换与赋值

```
package main

import (
	"fmt"
	"reflect"
)

//类型转换与赋值
type T struct {
	A int `newT:"AA"`
	B string `newT:"BB"`
}

type NewT struct {
	AA int
	BB string
}

func main() {
	t := T{
		A: 111,
		B: "hello",
	}
	tt := reflect.TypeOf(t)
	tv := reflect.ValueOf(t)

	newT := new(NewT)
	newTv := reflect.ValueOf(newT)

	for i := 0; i < tt.NumField(); i++ {
		field := tt.Field(i)
		newTTag := field.Tag.Get("newT")

		tValue := tv.Field(i)
		newTv.Elem().FieldByName(newTTag).Set(tValue)
	}

	fmt.Println(*newT)
}
```
#####  4、通过kind()处理不同分支
```
package main

import (
	"fmt"
	"reflect"
)

//通过kind()处理不同分支
func main() {
	a := 12345
	t := reflect.TypeOf(a)
	switch t.Kind() {
	case reflect.Int:
		fmt.Println("int")
	case reflect.String:
		fmt.Println("string")
	default:
		fmt.Println("Unknown")
	}
}
```
##### 5、判断实例是否实现了某接口
```
package main

import (
	"fmt"
	"reflect"
)

//判断实例是否实现了某接口
type IT interface {
	test1()
}

type T struct {
	A string
}

func (a *T) test1() {
	fmt.Println("test1")
}

func main() {
	t := new(T)
	ITF := reflect.TypeOf((*IT)(nil)).Elem()
	tt := reflect.TypeOf(t)
	res := tt.Implements(ITF)
	fmt.Println(res)
}
```

#### 参考资料
[https://golang.org/pkg/reflect/](https://golang.org/pkg/reflect/)

[https://github.com/astaxie/gopkg/tree/master/reflect](https://github.com/astaxie/gopkg/tree/master/reflect)

[https://segmentfault.com/a/1190000016230264](https://segmentfault.com/a/1190000016230264)