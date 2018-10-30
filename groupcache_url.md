## groupcache 源码分析（二）-- LRU
lru部分的代码在lru/lru.go文件中，它主要是封装了一系列lru算法相关的接口，供groupcahe进行缓存置换相关的调用。
它主要封装了下面几个接口：

```
// 创建一个Cache
func New(maxEntries int) *Cache
 
// 向Cache中插入一个KV
func (c *Cache) Add(key Key, value interface{})

// 从Cache中获取一个key对应的value
func (c *Cache) Get(key Key) (value interface{}, ok bool)

// 从Cache中删除一个key
func (c *Cache) Remove(key Key)

// 从Cache中删除最久未被访问的数据
func (c *Cache) RemoveOldest()

// 获取当前Cache中的元素个数
func (c *Cache) Len()

//清空当前Cache
func (c *Cache) Clear()

```
下面我们具体讲一下lru部分的全部代码。

```
//Cache是LRU的一个缓存区域，注意它并不是线程安全的
type Cache struct {
	//MaxEntries是Cache中实体的最大数量，0表示没有限制。
	MaxEntries int
	
	//OnEvicted是一个回调函数，进行Cache操作达到一定条件可能需要回调做一些处理工作。
	OnEvicted func(key Key, value interface{})
	
	//ll是引用container/list包中的双向链表，是一个链表指针
	ll    *list.List
	//cache是一个map，存放具体的k/v对，value是双向链表中的具体元素，也就是*Element
	cache map[interface{}]*list.Element
}

//key 是接口，可以是任意类型
type Key interface{}

//一个entry包含一个key和一个value，都是任意类型
type entry struct {
	key   Key
	value interface{}
}
```
New()函数传入Cache允许的最大实体数，返回一个初始化过的Cache指针。

```
// New creates a new Cache.
// If maxEntries is zero, the cache has no limit and it's assumed
// that eviction is done by the caller.
func New(maxEntries int) *Cache {
	return &Cache{
		MaxEntries: maxEntries,
		ll:         list.New(),
		cache:      make(map[interface{}]*list.Element),
	}
}
```
Add()函数传入一个K/V对，在Cache中添加这个实体。

```
// Add adds a value to the cache.
func (c *Cache) Add(key Key, value interface{}) {
	//如果cache为空，则重新初始化一下
	if c.cache == nil {
		c.cache = make(map[interface{}]*list.Element)
		c.ll = list.New()
	}
	//如果cache中有该key，则该实体移动到头部（保证最近访问在最前）
	//然后将kv对实体中的value值替换
	if ee, ok := c.cache[key]; ok {
		c.ll.MoveToFront(ee)
		ee.Value.(*entry).value = value
		return
	}
	//如果cache中没有该key，则将kv对封装成entry实体加入到双向链表头上
	ele := c.ll.PushFront(&entry{key, value})
	//让cache这个map的该key指向该kv对实体
	c.cache[key] = ele
	//检查Cache是否满，如果满，就移除最久未被访问的实体
	if c.MaxEntries != 0 && c.ll.Len() > c.MaxEntries {
		c.RemoveOldest()
	}
}
```
Get()函数传入一个key，返回一个是否有该key以及对应value。

```
// Get looks up a key's value from the cache.
func (c *Cache) Get(key Key) (value interface{}, ok bool) {
	if c.cache == nil {
		return
	}
	//如果有该key，记得要把该实体移动到头部，因为最近被访问了
	if ele, hit := c.cache[key]; hit {
		c.ll.MoveToFront(ele)
		return ele.Value.(*entry).value, true
	}
	return
}
```
Remove()函数传入一个key，将该key对应的kv对实体从cache中移除。

```
// Remove removes the provided key from the cache.
func (c *Cache) Remove(key Key) {
	if c.cache == nil {
		return
	}
	if ele, hit := c.cache[key]; hit {
		c.removeElement(ele)
	}
}
//执行具体移除的函数，供文件内部调用
func (c *Cache) removeElement(e *list.Element) {
	//双向链表中移除该实体
	c.ll.Remove(e)
	//cache这个map中也要删除对应的key
	kv := e.Value.(*entry)
	delete(c.cache, kv.key)
	//这里如果回调函数不为空，要执行一下回调函数
	if c.OnEvicted != nil {
		c.OnEvicted(kv.key, kv.value)
	}
}
```
RemoveOldest()函数删除最久未被访问的内容。

```
// RemoveOldest removes the oldest item from the cache.
func (c *Cache) RemoveOldest() {
	if c.cache == nil {
		return
	}
	//就是从双向链表的尾部删除一个元素
	ele := c.ll.Back()
	if ele != nil {
		c.removeElement(ele)
	}
}
```
Len（）函数返回Cache中元素数。

```
// Len returns the number of items in the cache.
func (c *Cache) Len() int {
	if c.cache == nil {
		return 0
	}
	//其实就是返回双向链表的元素数
	return c.ll.Len()
}
```


```
// Clear purges all stored items from the cache.
func (c *Cache) Clear() {
	if c.OnEvicted != nil {
		for _, e := range c.cache {
			kv := e.Value.(*entry)
			c.OnEvicted(kv.key, kv.value)
		}
	}
	c.ll = nil
	c.cache = nil
}
```

##### 参考资料：
<https://my.oschina.net/goskyblue/blog/656413>

<http://www.voidcn.com/article/p-kjpxomam-baw.html>