## os/exec 包详解

os/exec 包提供了执行linux命令的相关接口，主要有如下：

``` go
//这个函数主要是用来查询可执行二进制文件的路径
func LookPath(file string) (string, error)

//这个函数主要是用来初始化一个Cmd指针，Path和Args按参数初始化，其他字段执行默认初始化
//初始化后的Cmd用于后续执行run，start等函数
func Command(name string, arg ...string) *Cmd

//这个函数主要是执行*Cmd中的命令，把执行结果和错误合并到byte数组中
func (c *Cmd) CombinedOutput() ([]byte, error)

//这个函数主要是执行*Cmd中的命令，返回执行结果,不包含错误信息
func (c *Cmd) Output() ([]byte, error)

//这个函数主要是执行*Cmd中的命令，并且会等待命令执行完成，如果命令执行不成功，则返回错误信息
func (c *Cmd) Run() error

//这个函数主要是执行*Cmd中的命令，只是让命令开始执行，并不会等待命令执行完。
func (c *Cmd) Start() error

//这个函数主要是用于连接到命令启动时错误标准输出的管道，命令结束时，管道会自动关闭
//这里记得要保证在cmd命令结束前来读取内容，不然会读不到（一般结合Start与Wait来保证）
func (c *Cmd) StderrPipe() (io.ReadCloser, error)

//这个函数主要是用于连接到命令启动时标准输入的管道
func (c *Cmd) StdinPipe() (io.WriteCloser, error)

//这个函数主要是用于连接到命令启动时标准输出的管道，命令结束时，管道会自动关闭
func (c *Cmd) StdoutPipe() (io.ReadCloser, error)

//这个函数主要是等待*Cmd中的已开始执行的命令执行完成
func (c *Cmd) Wait() error

//这个函数主要是输出命令执行失败的错误信息
func (e *Error) Error() string

//这个函数主要是返回一个执行不成功命令的信息
func (e *ExitError) Error() string

//类似于Command，但是多了一个Context
//如果命令没有完成，但是context完成了，则可以终止命令的继续执行
func CommandContext(ctx context.Context, name string, arg ...string) *Cmd
```
#### 具体示例
LookPath()函数

``` go
package main

import (
	"bytes"
	"context"
	"errors"
	"fmt"
	"io/ioutil"
	"os/exec"
	"time"
)

//LookPath 实例
//执行结果:
//ps is at path:  /bin/ps
func lookPath(exe string) {
	path, err := exec.LookPath(exe)
	if err != nil {
		fmt.Println(err)
		return
	}

	fmt.Println(exe, "is at path: ", path)
}

//func Command && CombinedOutput 实例
//执行结果:
//exit status 1
//CombinedOutput Test:  mv: rename nofile to hasfile: No such file or directory
func combinedOutput() {
	arg := []string{"nofile", "hasfile"}
	cmd := exec.Command("mv", arg...)
	out, err := cmd.CombinedOutput()
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println("CombinedOutput Test: ", string(out))
}

//func Output 实例
//执行结果:
//exit status 1
//Output Test:
func outPut() {
	arg := []string{"nofile", "hasfile"}
	cmd := exec.Command("mv", arg...)
	out, err := cmd.Output()
	if err != nil {
		fmt.Println(err)
	}

	fmt.Println("Output Test: ", string(out))
}

//func Run 实例
//执行结果：
//Run Test: run cmd successfully!
func run() {
	arg := []string{"hello", "world"}
	cmd := exec.Command("echo", arg...)
	err := cmd.Run()
	if err != nil {
		fmt.Println("run cmd error: ", err)
		return
	}
	fmt.Println("Run Test: run cmd successfully!")
}

//func Start 实例
//执行结果：
//Start Test: start cmd successfully!(并不会等待10秒，而是立即返回立即打印)
func start() {
	cmd := exec.Command("sleep", "10")
	err := cmd.Start()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("Start Test: start cmd successfully!")
}

//func Wait 实例
//执行结果：
//Start Test: start run cmd!(立即打印)
//Start Test: finish run cmd!（10s后打印）
func wait() {
	cmd := exec.Command("sleep", "10")
	err := cmd.Start()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("Start Test: start run cmd!")
	err = cmd.Wait()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("Start Test: finish run cmd!")
}

//func StderrPipe 实例
//执行结果：
//exit status 64
//usage: mv [-f | -i | -n] [-v] source target
//       mv [-f | -i | -n] [-v] source ... directory
func stderrPipe() {
	cmd := exec.Command("mv", "hello world")
	stderr, err := cmd.StderrPipe()
	if err != nil {
		fmt.Println(err)
		return
	}
	err = cmd.Start()
	if err != nil {
		fmt.Println(err)
	}
	output, err := ioutil.ReadAll(stderr)
	if err != nil {
		fmt.Println(err)
		return
	}

	if err = cmd.Wait(); err != nil {
		fmt.Println(err) //exit status 64
	}

	fmt.Printf(string(output))
}

//func StdinPipe 实例
//执行结果：
//The output is: Hello World!
func stdinPipe() {
	var output bytes.Buffer
	cmd := exec.Command("cat")
	cmd.Stdout = &output
	stdin, err := cmd.StdinPipe()
	if err != nil {
		fmt.Println(err)
		return
	}
	if err = cmd.Start(); err != nil {
		fmt.Println(err)
	}
	stdin.Write([]byte("Hello World!"))
	stdin.Close()
	if err = cmd.Wait(); err != nil {
		fmt.Println(err)
	}

	fmt.Println("The output is: ", string(output.Bytes()))
}

//func StdoutPipe 实例
//执行结果：
//The output is: Hello World!
func stdoutPipe() {
	cmd := exec.Command("echo", "Hello World!")
	stdout, err := cmd.StdoutPipe()
	if err != nil {
		fmt.Println(err)
		return
	}
	if err = cmd.Start(); err != nil {
		fmt.Println(err)
	}
	output, _ := ioutil.ReadAll(stdout)
	if err = cmd.Wait(); err != nil {
		fmt.Println(err)
	}

	fmt.Printf("The output is: %s\n", output)
}

//func (e *Error) Error 实例
//执行结果：
//exec: "mv": 无法获取"Hello"的文件状态: 没有那个文件或目录
func eerror() {
	e := exec.Error{
		Name: "mv",
		Err:  errors.New("无法获取\"Hello\"的文件状态: 没有那个文件或目录"),
	}

	fmt.Println(e.Error())
}

//func (e *ExitError) Error 实例
//执行结果:
//exit status 64
func exiterror() {
	cmd := exec.Command("mv", "Hello World!")
	cmd.Run()
	exitError := exec.ExitError{cmd.ProcessState, nil}

	fmt.Println(exitError.Error())
}

//func CommandContext 实例
//执行结果：
//signal: killed
func contextCommand() {
	ctx, cancel := context.WithTimeout(context.Background(), 100*time.Millisecond)
	defer cancel()

	if err := exec.CommandContext(ctx, "sleep", "5").Run(); err != nil {
		// This will fail after 100 milliseconds. The 5 second sleep
		// will be interrupted.
		fmt.Println(err)
	}
}

func main() {
	lookPath("ps")
	combinedOutput()
	outPut()
	run()
	start()
	wait()
	stderrPipe()
	stdinPipe()
	stdoutPipe()
	eerror()
	exiterror()
	contextCommand()
}
```


#### 参考资料
[https://golang.org/pkg/os/exec/](https://golang.org/pkg/os/exec/)

[https://github.com/astaxie/gopkg/tree/master/os/exec](https://github.com/astaxie/gopkg/tree/master/os/exec)
