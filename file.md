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

```
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

```
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
func (b *Writer) Write(p []byte) (nn int, err error)
func (b *Writer) WriteByte(c byte) error
func (b *Writer) WriteRune(r rune) (size int, err error)
func (b *Writer) WriteString(s string) (int, error)
```
具体内容见下面代码以及注释：

```

```

#### 参考资料：
[https://golang.org/pkg/](https://golang.org/pkg/)

[https://github.com/astaxie/gopkg/tree/master/](https://github.com/astaxie/gopkg/tree/master/)

[https://colobu.com/2016/10/12/go-file-operations/](https://colobu.com/2016/10/12/go-file-operations/)

[https://www.jianshu.com/p/7790ca1bc8f6](https://www.jianshu.com/p/7790ca1bc8f6)

 
