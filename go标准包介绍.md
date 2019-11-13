go标准库包含很多包，详细见[https://golang.org/pkg/](https://golang.org/pkg/)

下面摘取部分比较常用的说明一下：

**strings包 ：** 

主要是处理字符串的一些函数集合，包括合并、查找、分割、比较、后缀检查、索引、大小写处理等等。
strings与bytes的函数接口功能基本一致。

**bytes包：**

bytes包提供了对字节切片进行读写操作的一系列函数。 字节切片处理的函数比较多，分为基本处理函数、比较函数、后缀检查函数、索引函数、分割函数、大小写处理函数和子切片处理函数等。
strings与bytes的函数接口功能基本一致。

**path包：**

path实现了对斜杠分隔的路径进行操作的函数。

**path/filepath包：**

filepath包实现了兼容各操作系统的文件路径操作函数，filepath包包含了path的函数，但是filepath可以适配不同的系统平台，该包更常用。

**os包：**

os 包提供了不依赖平台的操作系统函数接口，设计像Unix风格，但错误处理是go风格，当os包使用时，如果失败后返回错误类型而不是错误数量。
包括的实现有更改当前路径，更改权限等等linux中常用命令。

**os/exec包：**

exec包提供了执行自定义linux命令的相关实现。

**bufio包：**

bufio模块通过对io模块的封装，提供了数据缓冲功能，能够一定程度减少大块数据读写带来的开销。
在bufio各个组件内部都维护了一个缓冲区，数据读写操作都直接通过缓存区进行。当发起一次读写操作时，会首先尝试从缓冲区获取数据；只有当缓冲区没有数据时，才会从数据源获取数据更新缓冲。

**io/ioutil包：**

ioutil提供了对io包的封装函数。
主要是read的几种方式。

**log包：**

Go语言中log模块用于在程序中输出日志。
log模块提供了三类日志输出接口，Print、Fatal和Panic。Print是普通输出；Fatal是在执行完Print后，执行 os.Exit(1)；Panic是在执行完Print后调用panic()方法。log模块对每一类接口其提供了3中调用方式，分别是"Xxxx、 Xxxxln、Xxxxf"。
log可以实现定制。

**regexp包：**

regexp主要是提供了实现正则相关的功能函数。

**strconv包：**

strconv提供了字符串与基本类型的转换函数接口。

**time包：**

time包提供显示和计算时间用的函数。Go中时间处理依赖的数据类型: time.Time, time.Month, time.Weekday, time.Duration, time.Location。

**errors包：**

Go语言使用error类型来返回函数执行过程中遇到的错误，如果返回error值为nil，则表示未遇到错误，否则error会返回一个字符串。error是一个预定义标识符，代表一个Go语言內建的接口类型，任何自定义的错误类型都要实现Error接口函数。
errors可以实现定制。

**container/heap包：**

heap包实现了堆相关的一系列操作。

**container/list包：**

list包提供了双向链表的一系列操作。

**contain/ring包：**

ring包实现了ring环的一系列操作。

**context包：**
定义了Context类型，它跨API边界和进程之间传递截止时间，取消信号和其他请求范围的值，主要用于跟踪层层goruntine。

**database/sql/driver包：**

提供了数据库操作相关的函数实现。

**encoding/binary包：**

实现了数字和字节序列之间的转换以及整形变量的一些编码解码。

**encoding/json包：**

提供了json相关转换以及操作的实现。

**encoding/xml包：**

提供了xml格式相关转换以及操作的实现。

**flag包：**

提供基本的命令行操作的实现。

**fmt包：**

实现了格式化的标准输入输出。

**math包：**

提供了数学计算相关的实现。

**html/template包：**
主要实现了web开发中生成html的template的一些函数。

**net/http包：**
主要实现了http相关的一些操作函数。

**reflect包：**
主要是提供了反射特性的一些函数实现。

**sort包：**

提供了用于对切片和用户定义的集合进行排序的基元。

**sync包：**

实现多线程中锁机制以及其他同步互斥机制。

**unicode包：**

提供了对unicode编码进行属性判断的一些函数（比如是否数字？是否字母？等等）。

### 参考资料：

[https://golang.org/pkg/](https://golang.org/pkg/)

[https://github.com/astaxie/gopkg](https://github.com/astaxie/gopkg)

[https://studygolang.com/articles/15297](https://studygolang.com/articles/15297)

[https://studygolang.com/articles/15290](https://studygolang.com/articles/15290)
