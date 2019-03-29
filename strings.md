## strings包

strings包主要是提供string字符串的相关处理函数，主要包括：

#### 比较
```go
//比较a与b，相等返回0，a>b则1，a<b则-1
func Compare(a, b string) int

//字符串s和t比较，它们在全部小写的情况下，采用UTF8编码的底层的unicode是否一致
func EqualFold(s, t string) bool
```

#### 包含
```go
//这个函数主要是用来判断s中是否包含substr这个子串，如果包含返回true，否者返回false
func Contains(s, substr string) bool

//这个函数主要是用来判断s中是否包含chars中的字符中的任意字符，如果包含返回true，否者返回false
func ContainsAny(s, chars string) bool

//这个函数主要是用来判断s中是否包含rune类型的r字符，如果包含返回true，否者返回false
func ContainsRune(s string, r rune) bool

//这个函数主要是用来判断s中包含了多少个sep
func Count(s, substr string) int

//该函数主要判断s串中是否含有前缀prefix，如果包含，那么返回true，否则返回false
func HasPrefix(s, prefix string) bool

该函数主要判断s串中是否含有后缀prefix，如果包含，那么返回true，否则返回false
func HasSuffix(s, suffix string) bool
```

#### 分割
```go
//s按照一个空格或者多个连续的空格分割，返回分割之后的串数组，
//如果字符串没有空格或者只有一个空格，那么返回元素为去空格的s字符串
func Fields(s string) []string

//s的每一个字符传入函数f，如果f返回true，那么按照该字符进行分割（该字符不保留），
//继续下一个字符，以此类推直到最后，如果返回的都是为false或者s为空，那么将返回空的字符串slice
func FieldsFunc(s string, f func(rune) bool) []string

//该函数s根据sep分割，返回分割之后子字符串的slice，如果sep为空，那么每一个字符都分割
func Split(s, sep string) []string

//该函数s根据sep分割，返回分割之后子字符串的slice,和split一样，只是返回的子字符串保留sep，
//如果sep为空，那么每一个字符都分割
func SplitAfter(s, sep string) []string

//同SplitAfter差不多，只是n 表示需要分割的子字符串
//n > 0: 最多n个子字符串; 最后一个就是剩下未分割的子字符串.
//n == 0: 返回为0的字符串
//n < 0: 返回所有的子字符串，同SplitAfter
func SplitAfterN(s, sep string, n int) []string

//同Split差不多，只是n 表示需要分割的子字符串
//n > 0: 最多n个子字符串; 最后一个就是剩下未分割的子字符串.
//n == 0: 返回为0的字符串
//n < 0: 返回所有的子字符串，同Split
func SplitN(s, sep string, n int) []string
```

#### 索引
```go
//该函数主要判断sep串在s串中第一次出现的位置，如果不存在返回-1
func Index(s, substr string) int

//该函数主要判断chars集中任意的一个字符在s串中第一次出现的位置，如果不存在返回-1
func IndexAny(s, chars string) int

//该函数主要判断c字符在s串中第一次出现的位置，如果不存在返回-1
func IndexByte(s string, c byte) int

//该函数主要判断s中的每一个字符传入函数f，如果符合，那么返回该字符的位置，如果都不符合则返回-1
func IndexFunc(s string, f func(rune) bool) int

//该函数主要判断unicode r在s串中第一次出现的位置，如果不存在返回-1
func IndexRune(s string, r rune) int

//该函数主要判断sep串在s串中最后一次出现的位置，如果不存在返回-1
func LastIndex(s, substr string) int

//该函数主要判断chars集中任意的一个字符在s串中最后一次出现的位置，如果不存在返回-1
func LastIndexAny(s, chars string) int

//该函数主要判断c字符在s串中最后一次出现的位置，如果不存在返回-1
func LastIndexByte(s string, c byte) int

//该函数主要判断s中的每一个字符传入函数f，返回符合函数f的最后一个字符的位置，如果都不符合则返回-1
func LastIndexFunc(s string, f func(rune) bool) int
```
#### 拼接
```go
//该函数主要实现字符串slice的链接功能，把slice的每一个元素通过sep进行链接
func Join(a []string, sep string) string

//该函数以此读取s中的字符，传入mapping函数，然后返回的字符链接起来，
//就是字符串的每一个字符通过mapping函数的处理，最后返回处理好的字符串，
//如果处理不正确，那么就抛弃该字符
func Map(mapping func(rune) rune, s string) string

//该函数返回一个s的重复count字数的字符串
func Repeat(s string, count int) string
```

#### 替换
```go
//该函数实现在s中把old替换为new字符串，替换次数为n，如果n小于0，那么就全部替换
func Replace(s, old, new string, n int) string

//该函数实现在s中把old替换为new字符串，全部替换
func ReplaceAll(s, old, new string) string
```

#### 转换
```go
//该函数把s字符串里面的每个单词首字母转化为标题形式（大写）
func Title(s string) string

//该函数把s字符串里面的每个单词转化为小写（所有字符都转换，不仅仅是首字母）
func ToLower(s string) string

//该函数把s字符串里面的每个单词转化为小写，但是调用的是unicode.SpecialCase的ToLower方法
func ToLowerSpecial(c unicode.SpecialCase, s string) string

//该函数把s字符串里面的每个单词转化为标题体，其实和ToUpper一样的效果，
//但是有些语种的unicode,ToTitle和ToUpper效果不一样，但是我没试出来过，英语至少是一样的。
func ToTitle(s string) string

//该函数把s字符串里面的每个单词转化为标题体，但是调用的是unicode.SpecialCase的ToTitle方法
func ToTitleSpecial(c unicode.SpecialCase, s string) string

//该函数把s字符串里面的每个字符转化为大写（所有字符都转换）
func ToUpper(s string) string

//该函数把s字符串里面的每个单词转化为大写，但是调用的是unicode.SpecialCase的ToUpper方法
func ToUpperSpecial(c unicode.SpecialCase, s string) string
```

#### 裁剪
```go
//该函数把s字符串开头或者结尾里面包含字符集的字符全部过滤掉，返回过滤之后的字符串
func Trim(s string, cutset string) string

//该函数把s字符串里面开头和结尾部分传入f函数进行判断为真的字符过滤掉
func TrimFunc(s string, f func(rune) bool) string

//该函数把s字符串开头里面包含字符集的字符全部过滤掉，返回过滤之后的字符串
func TrimLeft(s string, cutset string) string

//该函数把s字符串里面开头部分传入f函数进行判断为真的字符过滤掉
func TrimLeftFunc(s string, f func(rune) bool) string

//该函数把s字符串结尾里面包含字符集的字符全部过滤掉，返回过滤之后的字符串
func TrimRight(s string, cutset string) string

//该函数把s字符串里面结尾部分传入f函数进行判断为真的字符过滤掉
func TrimRightFunc(s string, f func(rune) bool) string

//该函数把s字符串开头或者结尾里面空白符全部过滤掉，返回过滤之后的字符串
func TrimSpace(s string) string

//该函数把s字符串开头的prefix字符串去掉，如果不包含则不变
func TrimPrefix(s, prefix string) string

//该函数把s字符串结尾的suffix字符串去掉，如果不包含则不变
func TrimSuffix(s, suffix string) string
```

#### 参考资料
[https://golang.org/pkg/strings/#Compare](https://golang.org/pkg/strings/#Compare)

[https://github.com/astaxie/gopkg/tree/master/strings](https://github.com/astaxie/gopkg/tree/master/strings)