#  slice 原理

### Slice数据结构和原理
* 1：相对于数组，Slice的长度是动态可变的。如下：
```go
func CreatSlice() {
	s := make([]int, len(), cap())
	var s1 []int
}
func CreatArr() {
	var a [length]int
}
```
`可以很清楚的看到，数组的长度是在编译时静态计算的，并且数组无法在运行时动态扩缩容量的。`
* 2:在go的`/src/runtime/slice.go`中可以看到如下：
```go
type slice struct {
    array unsafe.Pointer
    len int
    cap int
}
```
<font color =red>此外无论是数组还是Slice都是按照值传递。并且！！！！goland中只有值传递！！！</font>

* 3:slice和数组的传递性能区别。
`talk is cheap, show code`
```go
func CallSlice(s []int) {

}
func CallArr(s [10000]int) {

}

func BenchMarkCallSlice(b *testing.B) {
    s := make([]int, 10000)
    for i := 0; i < b.N; i++ {
        CallSlice(s)
    }
}

func BenchMarkCallArr(b *testing.B) {
    var a [10000]int
    for i := 0; i < b.N; i++ {
        CallArr(a)
    }
}
```
>结果如下
![](https://img-blog.csdnimg.cn/f59c5a0555164089afabc2662c263f1b.png)


### Slice的实践
* 1:slice扩容过程中的坑。
 ```go
        var s []int
        fmt.Println(len(s),cap(s))//0,0
 ```
`这个就很简单的可以理解了，增加一个元素，容量和长度增加1个`
```go
	s = append(s, 0)
	fmt.Println(len(s), cap(s)) //1,1

	//s = append(s, 1, 2)
	//fmt.Println(len(s), cap(s)) //3,3

	s = append(s, 1)
        fmt.Println(len(s), cap(s)) //2,2
	s = append(s, 2)
	fmt.Println(len(s), cap(s)) //3,4
```
`但是这个注释掉的和下面的有什么区别？为什么长度一样，容量不一样？`
> 因为切片的每次扩容是前面扩容过的一倍，注释掉的代码就是一下append了两个，所以容量也是增加两个（内存优化）。


>
> 但是我们连续两次的append，第一次append后的容量为2，是1的二倍，然后再次append，再次翻倍，这也就是为什么为4。

最后：
```go
	for i := 3; i < 1025; i++ {
		s = append(s, i)
	}
	fmt.Println(len(s), cap(s)) //1025,1280
```
>为什么容量会为1280，而不是2048呢？talk is cheap, show code。我们可以在runtime/slice中找到growslice的方法，如下。
```go
newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.cap < 1024 {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}
```
>之后我们就能看到当容量大于等于1024情况下，增长倍数近似看为1.25倍。


## 扩容陷阱

##### 先看三个例子
>demo1
```go
func main() {
	var s []int
	for i := 0; i < 3; i++ {
		s = append(s, i)
	}
	modifySlice(s)
	fmt.Println(s) //[1024,1,2]
}

func modifySlice(s []int) {
	s[0] = 1024
}
```
-----------


>demo2
```go
func main() {
	var s []int
	for i := 0; i < 3; i++ {
		s = append(s, i)
	}
	modifySlice(s)
	fmt.Println(s) //[1024,1,2]
}

func modifySlice(s []int) {
	s = append(s, 2048)
	s[0] = 1024
}
```
 ------------------------------

>demo3
```go
func main() {
	var s []int
	for i := 0; i < 3; i++ {
		s = append(s, i)
	}
	modifySlice(s)
	fmt.Println(s) //[0 1 2]
}

func modifySlice(s []int) {
	s = append(s, 2048)
	s = append(s, 4096)
	s[0] = 1024
}

```
---------------------
>demo4
```go
func main() {
	var s []int
	for i := 0; i < 3; i++ {
		s = append(s, i)
	}
	modifySlice(s)
	fmt.Println(s) //[1024 1 2]
}

func modifySlice(s []int) {
	s[0] = 1024
	s = append(s, 2048)
	s = append(s, 4096)
}

```
<font color =red>重点来说demo2和demo3</font>
>首先demo2，在modifySlice的过程中，其实是传递的一个slice的一个结构体，虽然共享了底层的存储，但是在modifySlice里的操作，比如扩容，在外层是看不到的，所以也就是为什么2048无法打印出来。但是s[0]是直接操作了原有的结构体，所以，s[0]是可以更改的。
> 

>接下来是demo3,在第一次append的时候，len和cap都为4了，如果再次进行append会导致扩容，这个时候就导致两个s的底层存储空间发生变化。所以无法进行更改了。

## 常见问题
如下：
```go
	var s []int
	b, _ := json.Marshal(s)
    fmt.Println(string(b)) //null 
 ```

```go
	s := []int{}
	b, _ := json.Marshal(s)
	fmt.Println(string(b)) //[]
```
主要是和前端交互判断好是null还是[]，否则容易产生bug。
-----------------------------------------------
# Map

>在Slice中提到的是内层对数据的修改，在外层不可见。这点和Map有一些区别，如下：
> 
```go
func main() {
	m := make(map[int]int)
	modifyMap(m)
	fmt.Println(m) //map[1:1 2:2]
}

func modifyMap(m map[int]int) {
	m[1] = 1
	m[2] = 2
}
```
`因为map的底层是一个hmap的结构(在runtime里面可以看到，如下：),在使用make去创建的的时候，创建的不是一个结构体，而是一个指向hmap结构体的一个指针，所以在modifyMap中，传递的是指针，所以可以对外层进行修改。`

```go
type hmap struct {
	// Note: the format of the hmap is also encoded in cmd/compile/internal/reflectdata/reflect.go.
	// Make sure this stays in sync with the compiler's definition.
	count     int // # live cells == size of map.  Must be first (used by len() builtin)
	flags     uint8
	B         uint8  // log_2 of # of buckets (can hold up to loadFactor * 2^B items)
	noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
	hash0     uint32 // hash seed

	buckets    unsafe.Pointer // array of 2^B Buckets. may be nil if count==0.
	oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
	nevacuate  uintptr        // progress counter for evacuation (buckets less than this have been evacuated)

	extra *mapextra // optional fields
}
```

>接下来先看一下,B站一个up主对map的讲解大概了解一下原理。链接为：https://www.bilibili.com/video/BV1hv411x7we?p=4



>go语言中的实现主要是拉链法，如果需要对CPU缓存更有好的话建议采用开放地址法(但是这个是写在了runtime里面了，没办法修改)。
> 

><font color =red>map实际上的值，是一个指针，传的参数是指针，所以任何修改都会改变map的实际值。</font>

### 错误的用法
```go
	m := make(map[int]int)
	fmt.Print(&m[1])
```
* 是因为随着map的扩容，map的Key和Value都会发生改变，并且map保存的是值，会发生copy。


------------
#### map不会发生缩容
>go开发者中有句话为：Yes, maps that shrink permanently currently never get cleaned up after. As usual, the implementation challenge is with iterators.

Maps that shrink and grow repeatedly used to also cause leaks. That was #16070, fixed by CL 25049. I remember hoping when I started on that CL that the same mechanism would be useful for shrinking maps as well, but deciding it wouldn't. Sadly, I no longer remember why. If anyone wants to investigate this issue, I'd start by looking at that CL and thinking about whether that approach could be extended to shrinking maps.

The only available workaround is to make a new map and copy in elements from the old.大概意思是反复的扩容缩容会导致泄漏。


----------------

# Channel

#### 源码为：
```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dataqsiz elements
	elemsize uint16
	closed   uint32
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}
```

> 可以看到Channel底层有一个锁，这就导致在使用Channel的时候，最好只用于通知。不要进行值的传递，因为会发生Copy导致效率降低。
>
> 
>
--------------------------------
* 为什么Channel要在Runtime里面？导致无法进行优化？
>主要是因为Channel的调度和runtime是深刻绑定的。
> 如下图

![在这里插入图片描述](https://img-blog.csdnimg.cn/12eb4c63f9694fa9a85537f1e7375e79.png)

 
> 一个goroutine需要从Channel获取值，如果Channel里面有值，就直接取值，但是如果Channel内没有值，此goroutine就会一直等待，等到Channel有值。如果是多个，就会一起排队等待。直到另外的goroutine进行sendx。


------------------------
<font color =red>Channel是有锁结构</font>

<font color =red>Channel底层是ringbuffer</font>

<font color =red>Channel调用会出发调度</font>

<font color =red>Channel不适用高并发情况</font>

------------

## un/buffered channel
`buffered channel 会发生两次copy`

    * send goroutine -> buf
    * buf->receive goroutine


-----------------------

`unbuffered channel 发生一次copy`

    *send goroutine -> receive goroutine
------------------------


