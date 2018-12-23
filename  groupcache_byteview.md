## groupcache源码分析（五）-- byteview

byteview.go文件封装了一个string与byte[]的统一接口，也就是说用byteview提供的接口，可以屏蔽掉string与byte[]的不同，使用时可以不用考虑是string还是byte[]。

首先封装了一个ByteView结构体：

```
// A ByteView holds an immutable view of bytes.
// Internally it wraps either a []byte or a string,
// but that detail is invisible to callers.
//
// A ByteView is meant to be used as a value type, not
// a pointer (like a time.Time).
type ByteView struct {
	// If b is non-nil, b is used, else s is used.
	b []byte
	s string
}
```
然后提供了一系列方法：

```
// 返回长度
func (v ByteView) Len() int

// 按[]byte返回一个拷贝
func (v ByteView) ByteSlice() []byte

// 按string返回一个拷贝
func (v ByteView) String() string

// 返回第i个byte
func (v ByteView) At(i int) byte

// 返回ByteView的某个片断，不拷贝
func (v ByteView) Slice(from, to int) ByteView

// 返回ByteView的从某个位置开始的片断，不拷贝
func (v ByteView) SliceFrom(from int) ByteView

// 将ByteView按[]byte拷贝出来
func (v ByteView) Copy(dest []byte) int

// 判断2个ByteView是否相等
func (v ByteView) Equal(b2 ByteView) bool

// 判断ByteView是否和string相等
func (v ByteView) EqualString(s string) bool

// 判断ByteView是否和[]byte相等
func (v ByteView) EqualBytes(b2 []byte) bool

// 对ByteView创建一个io.ReadSeeker
func (v ByteView) Reader() io.ReadSeeker

// 读取从off开始的后面的数据，其实下面调用的SliceFrom，这是封装成了io.Reader的一个ReadAt方法的形式
func (v ByteView) ReadAt(p []byte, off int64) (n int, err error)

//向w流中写入v
func (v ByteView) WriteTo(w io.Writer) (n int64, err error)
```

所有函数其实都非常简单，就不一一解释各个方法了：

```
// Len returns the view's length.
//【返回一个view的长度】
func (v ByteView) Len() int {
    if v.b != nil {
        return len(v.b)
    }
    return len(v.s)
}

// ByteSlice returns a copy of the data as a byte slice.
//【获取一份[]byte类型的view值的拷贝】
func (v ByteView) ByteSlice() []byte {
    if v.b != nil {
        return cloneBytes(v.b)
    }
    return []byte(v.s)
}

// String returns the data as a string, making a copy if necessary.
//【上一个是返回[]byte类型，这里是string类型】
func (v ByteView) String() string {
    if v.b != nil {
        return string(v.b)
    }
    return v.s
}

// At returns the byte at index i.
//【返回第i个byte】
func (v ByteView) At(i int) byte {
    if v.b != nil {
        return v.b[i]
    }
    //【字符串索引获取到的值是byte类型】
    return v.s[i]
}

// Slice slices the view between the provided from and to indices.
//【返回从索引from到to的view的切分结果】
func (v ByteView) Slice(from, to int) ByteView {
    if v.b != nil {
        return ByteView{b: v.b[from:to]}
    }
    return ByteView{s: v.s[from:to]}
}

// SliceFrom slices the view from the provided index until the end.
//【相当于上面的to为len(b)】
func (v ByteView) SliceFrom(from int) ByteView {
    if v.b != nil {
        return ByteView{b: v.b[from:]}
    }
    return ByteView{s: v.s[from:]}
}

// Copy copies b into dest and returns the number of bytes copied.
//【拷贝一份view到dest】
func (v ByteView) Copy(dest []byte) int {
    if v.b != nil {
        return copy(dest, v.b)
    }
    return copy(dest, v.s)
}

// Equal returns whether the bytes in b are the same as the bytes in
// b2.
//【相等判断，具体比较的实现在下面】
func (v ByteView) Equal(b2 ByteView) bool {
    if b2.b == nil {
        return v.EqualString(b2.s)
    }
    return v.EqualBytes(b2.b)
}

// EqualString returns whether the bytes in b are the same as the bytes
// in s.
func (v ByteView) EqualString(s string) bool {
    if v.b == nil {
        //【如果b为nil，则比较s是否相等】
        return v.s == s
    }
    //【l为view的长度，实现在上面】
    l := v.Len()
    //【长度不同直接返回false】
    if len(s) != l {
        return false
    }
    //【判断[]byte中的每一个byte是否和string的每一个字符相等】
    for i, bi := range v.b {
        if bi != s[i] {
            return false
        }
    }
    return true
}

// EqualBytes returns whether the bytes in b are the same as the bytes
// in b2.
func (v ByteView) EqualBytes(b2 []byte) bool {
    //【b不为空，直接通过Equal方法比较】
    if v.b != nil {
        return bytes.Equal(v.b, b2)
    }
    l := v.Len()
    //【长度不等直接返回false】
    if len(b2) != l {
        return false
    }
    //【与上一个方法类似，比较字符串和[]byte每一个byte是否相等】
    for i, bi := range b2 {
        if bi != v.s[i] {
            return false
        }
    }
    return true
}

// Reader returns an io.ReadSeeker for the bytes in v.
//【返回值其实是bytes包或者strings包中的*Reader类型，这是个struct
// 实现了io.ReadSeeker等接口】
func (v ByteView) Reader() io.ReadSeeker {
    if v.b != nil {
        return bytes.NewReader(v.b)
    }
    return strings.NewReader(v.s)
}

// ReadAt implements io.ReaderAt on the bytes in v.
func (v ByteView) ReadAt(p []byte, off int64) (n int, err error) {
    if off < 0 {
        return 0, errors.New("view: invalid offset")
    }
    if off >= int64(v.Len()) {
        return 0, io.EOF
    }
    //【从off开始拷贝一份数据到p】
    n = v.SliceFrom(int(off)).Copy(p)
    if n < len(p) {
        err = io.EOF
    }
    return
}

// WriteTo implements io.WriterTo on the bytes in v.
//【将v写入w】
func (v ByteView) WriteTo(w io.Writer) (n int64, err error) {
    var m int
    if v.b != nil {
        m, err = w.Write(v.b)
    } else {
        m, err = io.WriteString(w, v.s)
    }
    if err == nil && m < v.Len() {
        err = io.ErrShortWrite
    }
    n = int64(m)
    return
}
```
#### 应用实例：
```
func TestByteView(t *testing.T) {
	for _, s := range []string{"", "x", "yy"} {
		for _, v := range []ByteView{of([]byte(s)), of(s)} {
			name := fmt.Sprintf("string %q, view %+v", s, v)
			if v.Len() != len(s) {
				t.Errorf("%s: Len = %d; want %d", name, v.Len(), len(s))
			}
			if v.String() != s {
				t.Errorf("%s: String = %q; want %q", name, v.String(), s)
			}
			var longDest [3]byte
			if n := v.Copy(longDest[:]); n != len(s) {
				t.Errorf("%s: long Copy = %d; want %d", name, n, len(s))
			}
			var shortDest [1]byte
			if n := v.Copy(shortDest[:]); n != min(len(s), 1) {
				t.Errorf("%s: short Copy = %d; want %d", name, n, min(len(s), 1))
			}
			if got, err := ioutil.ReadAll(v.Reader()); err != nil || string(got) != s {
				t.Errorf("%s: Reader = %q, %v; want %q", name, got, err, s)
			}
			if got, err := ioutil.ReadAll(io.NewSectionReader(v, 0, int64(len(s)))); err != nil || string(got) != s {
				t.Errorf("%s: SectionReader of ReaderAt = %q, %v; want %q", name, got, err, s)
			}
			var dest bytes.Buffer
			if _, err := v.WriteTo(&dest); err != nil || !bytes.Equal(dest.Bytes(), []byte(s)) {
				t.Errorf("%s: WriteTo = %q, %v; want %q", name, dest.Bytes(), err, s)
			}
		}
	}
}

// of returns a byte view of the []byte or string in x.
func of(x interface{}) ByteView {
	if bytes, ok := x.([]byte); ok {
		return ByteView{b: bytes}
	}
	return ByteView{s: x.(string)}
}
```
#### 参考资料：
<http://www.voidcn.com/article/p-kjpxomam-baw.html>
<https://www.cnblogs.com/cloudgeek/p/9693725.html>


