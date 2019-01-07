## Go 文件读写实例

go 文件读写主要有os、io/ioutil、bufio这几个包。

### io/ioutil

io/ioutil包中主要有这几个函数：

> func ReadAll(r io.Reader) ([]byte, error)

ReadAll()主要是用来是从一个打开的io.Reader中读取直到遇到error或EOF并返回读取的数据；成功的读取返回的err为nil，而不是EOF。因为ReadAll定义为从资源读取数据直到EOF，它不会将从r读取的EOF视为应该报告的错误。

io.Reader一般由os.Open()或者os.OpenFile()等进行打开。

> func ReadFile(filename string) ([]byte, error)

ReadFile()函数主要是用来从指定的filename文件中读取数据并返回文件的内容；成功的调用返回的err为nil，而不是EOF。因为ReadFile定义为从资源读取数据直到EOF，它不会将从r读取的EOF视为应该报告的错误。

> func WriteFile(filename string, data []byte, perm os.FileMode) error

WriteFile()函数将[]byte内容写入文件,如果content字符串中没有换行符的话，默认就不会有换行符。

如果文件不存在，则会按照perm的权限创建新文件；如果文件存在，原文件会被覆盖。

示例代码：

```go
package main

import (
	"io/ioutil"
	"os"
	"fmt"
)

func main() {
	
	//ReadAll
	file, err := os.OpenFile("filetest.txt", os.O_RDONLY, 0644)
	if err != nil {
		fmt.Println("Open file error!")
		return
	}
	defer file.Close()

	output, err := ioutil.ReadAll(file)
	if err != nil {
		fmt.Println("Read file error!")
		return
	}
	fmt.Println(string(output))

	//ReadFile
	output, err = ioutil.ReadFile("filetest.txt")
	if err != nil {
		fmt.Println("Read file error!")
		return
	}
	fmt.Println(string(output))

	//WriteFile
	context := "Write Test Successfully!"
	if ioutil.WriteFile("filetest.txt", []byte(context), 0644) != nil {
		fmt.Println("Write file error!")
	}else {
		fmt.Println("Write file successfully!")
	}
}
```
### os
os包中主要有以下函数：

> func Create(name string) (*File, error)
   
> func Open(name string) (*File, error)
   
> func OpenFile(name string, flag int, perm FileMode) (*File, error)
   
> func (f *File) Close() error
   
> func (f *File) Read(b []byte) (n int, err error)
   
> func (f *File) ReadAt(b []byte, off int64) (n int, err error)
   
> func (f *File) Write(b []byte) (n int, err error)
   
> func (f *File) WriteAt(b []byte, off int64) (n int, err error)
   
> func (f *File) WriteString(s string) (n int, err error)

os包中还有很多文件重命名、删除、软连接、目录等的操作函数，具体见官方文档。

上面列出的读写函数具体内容直接见代码以及代码注释：

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	//Open
	//以只读方式打开文件，若文件不存在则报错
	file1, err := os.Open("filetest.txt")
	if err != nil {
		fmt.Println("Open file error!")
		return
	}
	fmt.Println("Open file successfully!")
	defer file1.Close()

	//Create
	//创建一个新文件，若存在则覆盖
	file2, err := os.Create("hello.txt")
	if err != nil {
		fmt.Println("Create file error!")
		return
	}
	fmt.Println("Create file successfully!")
	defer file2.Close()

	//OpenFile
	//打开一个文件，打开方式以及文件权限由参数控制
	/* 打开方式可以由一下一种或多种组合，具体见下面注释
	  //Exactly one of O_RDONLY, O_WRONLY, or O_RDWR must be specified.
        O_RDONLY int = syscall.O_RDONLY // open the file read-only.
        O_WRONLY int = syscall.O_WRONLY // open the file write-only.
        O_RDWR   int = syscall.O_RDWR   // open the file read-write.
      // The remaining values may be or'ed in to control behavior.
        O_APPEND int = syscall.O_APPEND // append data to the file when writing.
        O_CREATE int = syscall.O_CREAT  // create a new file if none exists.
        O_EXCL   int = syscall.O_EXCL   // used with O_CREATE, file must not exist.
        O_SYNC   int = syscall.O_SYNC   // open for synchronous I/O.
        O_TRUNC  int = syscall.O_TRUNC  // if possible, truncate file when opened.
    */
    file3, err := os.OpenFile("filetest.txt", os.O_RDWR | os.O_APPEND | os.O_APPEND, 0777)
    if err != nil {
    	fmt.Println("Open file error!")
    	return
    }
    fmt.Println("Open file successfully!")
    defer file3.Close()

    //Read
    //如果out容量小于文件内容，则只读取out容量部分内容
    //如果out容量大于文件内容，则读取全部文件内容
    //如果文件为空，则出错
    out1 := make([]byte, 20)
    _, err = file1.Read(out1)
    if err != nil {
    	fmt.Println("Read file error!", err)
    	return
    }
    fmt.Println(string(out1))

    //ReadAt
    //如果out容量小于文件开始读取位置后的内容，则截取out容量部分内容
    //如果out容量大于文件开始读取位置后的内容或者文件为空或者开始读取位置超过了文件内容，则出错
    out2 := make([]byte, 9)
    _, err = file3.ReadAt(out2, 2)
    if err != nil {
    	fmt.Println("ReadAt file error!", err)
    	return
    }
    fmt.Println(string(out2))

    //Write
    //将[]byte中内容写入文件
    //是覆盖文件内容还是在后面添加内容取决于打开文件的方式
    in := []byte("haha beijing\n")
    _, err = file2.Write(in)
    if err != nil {
    	fmt.Println("Write file error!")
    	return
    }
    fmt.Println("Write file successfully!")

    //WriteAt
    //将[]byte写入文件的指定位置
    //会覆盖掉写入位置以后的内容（只覆盖要写入的数量的内容）
    in2 := []byte("hello ")
    _, err = file2.WriteAt(in2, 5)
    if err != nil {
    	fmt.Println("WriteAt file error!")
    	return
    }
    fmt.Println("WriteAt file successfully!")

    //WriteString
    //将string内容写入到文件
    //是覆盖文件内容还是在后面添加内容取决于打开文件的方式
    _, err = file3.WriteString("nihao beijing\n")
    if err != nil {
    	fmt.Println("WriteString file error!")
    	return
    }
    fmt.Println("WriteString file successfully!")

}
```

### bufio
bufio包主要是包装了io.Reader以及io.Writer然后生成两个类型：Reader以及Writer，并且衍生出了一系列读写的函数，主要如下：

```
type Reader

func NewReader(rd io.Reader) *Reader
func NewReaderSize(rd io.Reader, size int) *Reader
func (b *Reader) Read(p []byte) (n int, err error)
func (b *Reader) ReadByte() (c byte, err error)
func (b *Reader) ReadBytes(delim byte) (line []byte, err error)
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
func (b *Reader) ReadRune() (r rune, size int, err error)
func (b *Reader) ReadSlice(delim byte) (line []byte, err error)
func (b *Reader) ReadString(delim byte) (line string, err error)


type Writer

func NewWriter(wr io.Writer) *Writer
func NewWriterSize(wr io.Writer, size int) *Writer
func (b *Writer) Flush() error
func (b *Writer) Write(p []byte) (nn int, err error)
func (b *Writer) WriteByte(c byte) error
func (b *Writer) WriteRune(r rune) (size int, err error)
func (b *Writer) WriteString(s string) (int, error)
```
具体内容见下面代码以及注释：

```go
package main

import (
	"fmt"
	"bufio"
	"bytes"
	"os"
)

func main() {
	
	rb := bytes.NewBuffer([]byte("hello beijing!"))
	rb1, err := os.OpenFile("filetest.txt", os.O_RDWR | os.O_APPEND | os.O_APPEND, 0777)
    if err != nil {
    	fmt.Println("Open file error!")
    	return
    }
    fmt.Println("Open file successfully!")
    defer rb1.Close()

	//NewReader
	//创建支持缓存读取的具有缺省长度缓冲区的Reader对象，
	//Reader对象会从底层的io.Reader接口读取尽量多的数据进行缓存。
	//参数是io.Reader，通常是打开的file句柄.
	r1 := bufio.NewReader(rb)
	
	//NewReaderSize
	//创建的支持缓存读取的具有指定长度缓冲区的Reader对象，
	//Reader对象会从底层的io.Reader接口读取尽量多的数据进行缓存。
	//参数是io.Reader，通常是打开的file句柄.
	r2 := bufio.NewReaderSize(rb1, 10)

	//Read
	//读取数据存放到p中，返回已读取的字节数。
	//因为最多只调用底层的io.Reader一次，所以返回的n可能小于len(p)。
	//在字节流结束时，n会为0并且err为io.EOF。
	//如果len(out)<文件内容，则读取len(out)数量
	//如果len(out)>文件内容, 则读取文件全部内容
	//如果文件空，则出错
	out1 := make([]byte, 100)
	_, err = r1.Read(out1)
	if err != nil {
		fmt.Println("Read file error!")
		return
	}
	fmt.Println(string(out1))

	//ReadByte
	//读取并返回一个字节。如果没有字节可读，则返回错误。
	bout, err := r2.ReadByte()
	if err != nil {
		fmt.Println("ReadByte file error!", err)
		return
	}
	fmt.Println(string(bout))

	//ReadBytes
	//ReadBytes读取数据直到delim第一次出现，返回读取的字节序列（包括delim）。
	//如果ReadBytes在读到第一个delim之前出错，它返回已读取的数据和那个错误（通常是io.EOF）。
	//只有当返回的数据以delim结尾时，返回的err才为空值。
	//返回的是数据的拷贝
	rb3 := bytes.NewBuffer([]byte("hello beijing!"))
	r3 := bufio.NewReader(rb3)
	bouts, err := r3.ReadBytes('!')
	if err != nil {
		fmt.Println("ReadBytes file error!")
		return
	}
	fmt.Println(string(bouts))

	//ReadLine
	//ReadLine试图返回一行，不包括结尾的回车字符。
	//如果一行太长了（超过缓冲区长度），isPrefix会设置为true并且只返回前面的数据，剩余的数据会在以后的调用中返回。
	//当返回最后一行数据时，isPrefix会设置为false。返回的字节切片只在下一次调用ReadLine前有效。
	//ReadLine或者返回一个非空的字节切片或者返回一个错误，但它们不会同时返回。
	rb4 := bytes.NewBuffer([]byte("hello beijing\nnihao beijing\n"))
	r4 := bufio.NewReader(rb4)
	line, prefix, err := r4.ReadLine()
	if err != nil {
		fmt.Println("ReadLine file error!")
		return
	}
	fmt.Println(string(line), prefix, err)

	//ReadRune
	//ReadRune读取一个UTF-8编码的字符，并将其对应的Unicode编码和所占字节数返回。
	//如果编码错误，ReadRune只读取一个字节并返回unicode.ReplacementChar(U+FFFD)和长度1。
	rb5 := bytes.NewBuffer([]byte("你好世界"))
	r5 := bufio.NewReader(rb5)
	r, size, err := r5.ReadRune()
	if err != nil {
		fmt.Println("ReadRune file error!")
		return
	}
	fmt.Println(string(r), size, err)

	//ReadSlice
	//ReadSlice读取数据直到delim出现，并返回读取数据的字节切片。
	//下次读取数据时返回的切片会失效。
	//如果ReadSlice在查找到delim之前遇到错误，它返回读取的所有数据和那个错误（通常是io.EOF）。
	//如果缓冲区满时也没有查找到delim，则返回ErrBufferFull错误。
	//因为ReadSlice返回的数据会在下次I/O操作时被覆盖，大多数调用者应该使用ReadBytes或者ReadString。
	//只有当line不以delim结尾时，ReadSlice才会返回非空err。
	rb6 := bytes.NewBuffer([]byte("1234$5678"))
	r6 := bufio.NewReader(rb6)
	slice, err := r6.ReadSlice('$')
	if err != nil {
		fmt.Println("ReadSlice file error!")
		return
	}
	fmt.Println(string(slice), err)

	//ReadString
	//ReadString读取数据直到delim第一次出现，返回一个包含delim的字符串。
	//如果ReadString在读取到delim前遇到错误，它返回已读字符串和那个错误（通常是io.EOF）。
	//只有当返回的字符串以delim结尾时，ReadString才返回空err。
	//基本等同于ReadBytes，只不过返回的是string而不是[]byte。
	rb7 := bytes.NewBuffer([]byte("hello, world"))
	r7 := bufio.NewReader(rb7)
	str, err := r7.ReadString(',')
	if err != nil {
		fmt.Println("ReadString file error!")
		return
	}
	fmt.Println(str, err)

	//NewWriter + Flush + Write

	//NewWrite
	//创建支持缓存写的具有缺省长度缓冲区的Writer对象，
	//Writer对象会将缓存的数据批量写入底层的io.Writer接口

	//Flush把缓冲区中的数据写入底层的io.Writer

	//Write把p写入缓冲区，返回已写入的字节数。
	//如果nn小于len(p)，则同时返回一个错误说明原因。

	wb1 := bytes.NewBuffer(nil)
	w1 := bufio.NewWriter(wb1)
	w1.Write([]byte("hello, world"))
	fmt.Println(len(wb1.Bytes()), string(wb1.Bytes()))
	w1.Flush()
	fmt.Println(len(wb1.Bytes()), string(wb1.Bytes()))

	//NewWriterSize + WriteByte

	//NewWriterSize
	//创建支持缓存写的具有指定长度缓冲区的Writer对象，
	//Writer对象会将缓存的数据批量写入底层的io.Writer接口
	//如果缓冲区很小，缓冲区满后会自动Flush。

	//WriteByte
	//写入一个字节

	wb2 := bytes.NewBuffer(nil)
	w2 := bufio.NewWriterSize(wb2, 1024)
	w2.WriteByte('a')
	fmt.Println(len(wb2.Bytes()), string(wb2.Bytes()))
	w2.Flush()
	fmt.Println(len(wb2.Bytes()), string(wb2.Bytes()))

	//WriteRune
	//WriteRune以UTF-8编码写入一个Unicode字符，返回写入的字节数和错误。
	wb3 := bytes.NewBuffer(nil)
	w3 := bufio.NewWriter(wb3)
	w3.WriteRune('好')
	w3.Flush()
	fmt.Println(string(wb3.Bytes()))

	//WriteString
	//WriteString写入一个字符串，返回写入的字节数。
	//如果返回的字节数小于len(s)，则同时返回一个错误说明原因。
	wb4 := bytes.NewBuffer(nil)
	w4 := bufio.NewWriter(wb4)
	w4.WriteString("hello, shandong!")
	w4.Flush()
	fmt.Println(string(wb4.Bytes()))


}
```

#### 参考资料：
[https://golang.org/pkg/](https://golang.org/pkg/)

[https://github.com/astaxie/gopkg/tree/master/](https://github.com/astaxie/gopkg/tree/master/)

[https://colobu.com/2016/10/12/go-file-operations/](https://colobu.com/2016/10/12/go-file-operations/)

[https://www.jianshu.com/p/7790ca1bc8f6](https://www.jianshu.com/p/7790ca1bc8f6)

 
