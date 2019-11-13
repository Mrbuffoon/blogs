## groupcache源码分析（三）-- consistenthash
consistenthash.go文件中是consistenthash模块的代码，这主要是提供了一致性hash的一些接口。一致性hash算法，通常是用在查找一个合适的下载节点时，使负载更平均，同时也使得某个节点故障不会导致大量的重新映射成本，要了解一致性hash原理请详见：<https://www.cnblogs.com/lpfuture/p/5796398.html>

该部分主要封装了以下这几个接口：

```go
//创建一个HashMap
func New(replicas int, fn Hash) *Map

//判断HashMap是否为空
func (m *Map) IsEmpty() bool

//向HashMap中增加几个key
func (m *Map) Add(keys ...string)

//返回距离给定key最近的实体
func (m *Map) Get(key string) string
```

下面具体讲解一下每个函数。

首先定义了一个函数，用于将key值 Hash成32位整数，然后定义了Map结构体用于存放Hash后的结果，其中hash是上面的hash函数，Map结构中replicas的含义是增加虚拟桶，使数据分布更加均匀，keys存放hash后的结果，并且经过了排序，其实就是一致性hash圆环，hashMap就是存放具体的对应，将key对应上hash后的32位整数。

```go

type Hash func(data []byte) uint32

type Map struct {
	hash     Hash
	replicas int
	keys     []int // Sorted
	hashMap  map[int]string
}
```
New（）创建一个Map结构。这里注意如果hash函数为nil，则默认Hash函数为crc32库的ChecksumIEEE函数。

```go
func New(replicas int, fn Hash) *Map {
	m := &Map{
		replicas: replicas,
		hash:     fn,
		hashMap:  make(map[int]string),
	}
	if m.hash == nil {
		m.hash = crc32.ChecksumIEEE
	}
	return m
}
```
IsEmpty()函数返回MAP是否为空。

```go
// Returns true if there are no items available.
func (m *Map) IsEmpty() bool {
	return len(m.keys) == 0
}
```
Add（）函数增加一些key到Map中。

```go
// Adds some keys to the hash.
func (m *Map) Add(keys ...string) {
	//遍历要增加的key集合
	for _, key := range keys {
		//这里是为了产生虚拟节点
		for i := 0; i < m.replicas; i++ {
		   //将i与key拼接之后再进行Hash
			hash := int(m.hash([]byte(strconv.Itoa(i) + key)))
			//keys结构体增加key
			m.keys = append(m.keys, hash)
			//hashMap中增加key与Hash结果的映射
			m.hashMap[hash] = key
		}
	}
	sort.Ints(m.keys)
}
```
Get（）函数根据key找到对应的节点

```go
// Gets the closest item in the hash to the provided key.
func (m *Map) Get(key string) string {
	if m.IsEmpty() {
		return ""
	}
	//先找到key对应的Hash值
	hash := int(m.hash([]byte(key)))

	//找到key对应hash值之后的最近的一个节点
	// Binary search for appropriate replica.
	idx := sort.Search(len(m.keys), func(i int) bool { return m.keys[i] >= hash })
    //如果idx是keys的最后一个元素，则定向到第一个，因为模拟的是环
	// Means we have cycled back to the first replica.
	if idx == len(m.keys) {
		idx = 0
	}
   //返回对应的节点
	return m.hashMap[m.keys[idx]]
}

```

####简单应用举例：
```go
package main

import (
	"fmt"

	"github.com/golang/groupcache/consistenthash"
)

func main() {
	c := consistenthash.New(70, nil)
	c.Add("A", "B", "C", "D", "E")
	for _, key := range []string{"what", "nice", "what", "nice", "good", "yes!"} {
		fmt.Printf("%s -> %s\n", key, c.Get(key))
	}
}

// Expect output
// -------------
// what -> C
// nice -> A
// what -> C
// nice -> A
// good -> D
// yes! -> E
```

#### 参考资料：

<https://www.cnblogs.com/lpfuture/p/5796398.html>

<https://my.oschina.net/goskyblue/blog/656413>

<http://www.apepro.com/2016/08/09/groupcache-analysis-1/>