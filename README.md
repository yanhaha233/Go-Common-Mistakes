Go 语言中常见的学习错误，根据 Go 的 50 度灰整理。
原文：[50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)

### 一、初级

#### 1.开大括号不能放在单独的一行

```go
// 错误示例
package main

import "fmt"

func main()
{   //error, can't have the opening brace on a separate line
    fmt.Println("Hello World!")
}
```

#### 2.未使用的变量

```go
// 代码中不能存在未使用的变量
package main

var gvar int //not an error

func main() {
    var one int   //error, unused variable
    two := 2      //error, unused variable
    var three int //error, even though it's assigned 3 on the next line
    three = 3
}
```

#### 3.未使用的 Imports

如果你引入一个包，而没有使用其中的任何函数、接口、结构体或者变量的话，代码将会编译失败。

如果你真的需要引入的包，你可以添加一个下划线标记符，`_`，来作为这个包的名字，从而避免编译失败。下滑线标记符用于引入，但不使用。

#### 4.简式的变量声明仅可以在函数内部使用

```go
package main

myvar := 1 //error

func main{

}
```

#### 5.使用简式声明重复声明变量

你不能在一个单独的声明中重复声明一个变量，但在多变量声明中这是允许的，其中至少要有一个新的声明变量。
重复变量需要在相同的代码块内，否则你将得到一个隐藏变量。

Fails:

```go
package main

func main(){
    one := 1
    one := 2 //error
}
```

Works：

```go
package main

func main(){
    one := 1
    one,two := 2,3
}
```

#### 6.偶然的变量隐藏 Accidental Variable Shadowing

短式变量声明的语法如此的方便，很容易让人把它当成一个正常的分配操作。

```go
package main

import "fmt"

func main() {
    x := 1
    fmt.Println(x)     //prints 1
    {
        fmt.Println(x) //prints 1
        x := 2
        fmt.Println(x) //prints 2
    }
    fmt.Println(x)     //prints 1 (bad if you need 2)
}
```

#### 7.不使用显式类型，无法使用 nil 来初始化变量

`nil`标志符用于表示 interface,func,map,slice 和 channel 的零值。如果不指定变量的类型，则会报错，因为无法猜测它具体的类型。

Fails:

```go
package main

func main(){
    var x = nil // error

    _ = x
}
```

Works:

```go
package main

func main(){
    var x interface{} = nil

    _ = x
}
```

#### 8.使用 nil Slices and Maps

在一个`nil`的 slice 中添加元素是没问题的，但对 map 做同样的事时将会生成一个运行的 panic。

Fails:

```go
package main

func main(){
    var m map[string]int
    m["one"] = 1 //error
}
```

Works:

```go
package main

func main(){
    var s []int
    s = append(s,1)
}
```

#### 9.Map 的容量

你可以在 map 创建时指定它的容量，但你无法在 map 上使用 cap()函数。

Fails:

```go
package main

func main(){
    m := make(map[string]int,99)
    cap(m) //error
}
```

#### 10.字符串不会为 nil

Fails:

```go
package main

func main(){
    var x string = nil //error

    if x == nil{
        x = "default"
    }
}
```

Works:

```go
package main

func main(){
    var x string  // defaults to ""

    if x == ""{
        x = "default"
    }
}
```

#### 11.Array 函数的参数

Go 中的数组是数值，因此当你向函数中传递数组时，函数会得到原始数组数据的一份复制。

```go
package main

func main(){
    x := [3]int{1,2,3}

    func(arr [3]int){
        arr[0] = 7
        fmt.Println(arr) // prints [7 2 3]
    }(x)

    fmt.Println(x) // prints[1 2 3] (not ok if you need [7 2 3])
}
```

如果你需要更新原始数组的数据，你可以使用数组指针类型。

```go
package main

func main(){
    x := [3]int{1,2,3}

    func(arr *[3]int){
        arr[0] = 7
        fmt.Println(arr) // prints [7 2 3]
    }(&x)

    fmt.Println(x) // prints[7 2 3]
}
```

另一个选择是使用 slice。即使你的函数得到了 slice 变量的一份拷贝，它依旧会参照原始的数据。

```go
package main

import "fmt"

func main(){
    x := []int{1,2,3}

    func(arr []int){
        arr[0] = 7
        fmt.Println(arr) // prints [7 2 3]
    }(x)

    fmt.Println(x) // prints [7 2 3]
}

```

#### 12.在 Slice 和 Array 使用 range

Go 中的“range”语法不太一样。它会得到两个值：第一个值是元素的索引，而另一个值是元素的数据。

Bad:

```go
package main

import "fmt"

func main(){
    x := []string{"a","b","c"}

    for v := range x {
        fmt.Println(v) // prints 0,1,2
    }
}
```

Good:

```go
package main

import "fmt"

func main(){
    x := []string{"a","b","c"}

    for _, v := range x{
        fmt.Println(v) // prints a,b,c
    }
}
```

#### 13.Slices 和 Arrays 是一维的

使用“独立”slice 来创建一个动态的多维数组需要两步。首先，你需要创建一个外部的 slice。然后，你需要分配每个内部的 slice。内部的 slice 相互之间独立。你可以增加减少它们，而不会影响其他内部的 slice。

```go
package main

func main(){
    x := 2
    y := 4
    table := make([][]int,x)
    for i:= range table{
        table[i] = make([]int,y)
    }
}
```

使用“共享数据”slice 的 slice 来创建一个动态的多维数组需要三步。首先，你需要创建一个用于存放原始数据的数据“容器”。然后，你再创建外部的 slice。最后，通过重新切片原始数据 slice 来初始化各个内部的 slice。

```go
package main

import "fmt"

func main() {
	h,w := 2,4

	raw := make([]int,h*w)

	for i := range raw{
		raw[i] = i
	}
    fmt.Println(raw,&raw[4])
    //prints:[0 1 2 3 4 5 6 7] <ptr_addr_x>

	table := make([][]int,h)
	for i := range table{
		table[i] = raw[i*w:i*w + w]
	}

    fmt.Println(table,&table[1][0])
    //prints:[[0 1 2 3] [4 5 6 7]] <ptr_addr_x>
}
```

#### 14.访问不存在的 Map Keys

这对于那些希望得到“nil”标示符的开发者而言是个技巧（和其他语言中做的一样）。如果对应的数据类型的“零值”是“nil”，那返回的值将会是“nil”，但对于其他的数据类型是不一样的。检测对应的“零值”可以用于确定 map 中的记录是否存在，但这并不总是可信（比如，如果在二值的 map 中“零值”是 false，这时你要怎么做）。检测给定 map 中的记录是否存在的最可信的方法是，通过 map 的访问操作，检查第二个返回的值。

Bad:

```go
package main

import "fmt"

func main() {
	x := map[string]string{"one":"a","two":"","three":"c"}

	if v := x["two"];v == ""{
		fmt.Println("no entry")
	}
}
```

Good:

```go
package main

import "fmt"

func main() {
	x := map[string]string{"one":"a","two":"","three":"c"}

	if _,ok := x["two"];!ok{
		fmt.Println("no entry")
	}
}
```

#### 15.Strings 无法修改

尝试使用索引操作来更新字符串变量中的单个字符将会失败。string 是只读的 byte slice（和一些额外的属性）。如果你确实需要更新一个字符串，那么使用 byte slice，并在需要时把它转换为 string 类型。

Fails:

```go
package main

import "fmt"

func main() {
	x := "text"
	x[0] = 'T'

	fmt.Println(x)
}

// cannot assign to x[0]
```

Works:

```go
package main

import "fmt"

func main() {
	x := "text"
	xs := []byte(x)
    xs[0] = 'T'

    fmt.Println(string(xs))
    //prints Text
}

```

需要注意的是：这并不是在文字 string 中更新字符的正确方式，因为给定的字符可能会存储在多个 byte 中。如果你确实需要更新一个文字 string，先把它转换为一个 rune slice。即使使用 rune slice，单个字符也可能会占据多个 rune，比如当你的字符有特定的重音符号时就是这种情况。这种复杂又模糊的“字符”本质是 Go 字符串使用 byte 序列表示的原因。

#### 16.String 和 Byte Slice 之间的转换

当你把一个字符串转换为一个 `byte slice`（或者反之）时，你就得到了一个原始数据的完整拷贝。这和其他语言中 cast 操作不同，也和新的 `slice` 变量指向原始 `byte` slice 使用的相同数组时的重新 slice 操作不同。

Go 在`[]byte` 到 `string` 和 `string` 到`[]byte` 的转换中确实使用了一些优化来避免额外的分配（在 todo 列表中有更多的优化）。

第一个优化避免了当`[]byte keys`用于在`map[string]`集合中查询时的额外分配:`m[string(key)]`。

第二个优化避免了字符串转换为`[]byte` 后在 `for range` 语句中的额外分配：`for i,v := range []byte(str) {...}`。

#### 17.String 和索引操作

字符串上的索引操作返回一个 byte 值，而不是一个字符。

```go
package main

import "fmt"

func main() {
	x := "text"
	fmt.Println(x[0])  // print 116
	fmt.Printf("%T",x[0]) // prints uint8
}
```

#### 18.字符串不总是 UTF8 文本

字符串的值不需要是 UTF8 的文本。它们可以包含任意的字节。只有在 string literal 使用时，字符串才会是 UTF8。即使之后它们可以使用转义序列来包含其他的数据。

为了知道字符串是否是 UTF8，你可以使用“unicode/utf8”包中的 ValidString()函数。

```go
package main

import (
	"fmt"
	"unicode/utf8"
)

func main() {
	data1 := "ABC"
	fmt.Println(utf8.ValidString(data1)) //true

	data2 := "A\xfeC"
	fmt.Println(utf8.ValidString(data2)) //false
}
```

#### 19.字符串的长度

python 中：

```python
data = u'♥'
print(len(data)) #prints: 1
```

Go 中:

```go
package main
import "fmt"
func main() {
    data := "♥"
    fmt.Println(len(data)) //prints: 3
}
```

内建的 len()函数返回 byte 的数量，而不是像 Python 中计算好的 unicode 字符串中字符的数量。

要在 Go 中得到相同的结果，可以使用“unicode/utf8”包中的 RuneCountInString()函数。

```go
package main
import (
    "fmt"
    "unicode/utf8"
)
func main() {
    data := "♥"
    fmt.Println(utf8.RuneCountInString(data)) //prints: 1
}
```

理论上说 RuneCountInString()函数并不返回字符的数量，因为单个字符可能占用多个 rune。

```go
package main
import (
    "fmt"
    "unicode/utf8"
)
func main() {
    data := "é"
    fmt.Println(len(data))                    //prints: 3
    fmt.Println(utf8.RuneCountInString(data)) //prints: 2
}
```

#### 20.在多行的 Slice、Array 和 Map 语句中遗漏逗号

Fails:

```go
package main

func main() {
	x:=[]int{
		1,
		2 //error
	}
	_ = x
}
```

Works:

```go
package main

func main() {
	x := []int{
		1,
		2,
	}
	x = x

	y := []int{3,4,} // no error
	y = y
}

```

#### 21.log.Fatal 和 log.Panic 不仅仅是 Log

Logging 库一般提供不同的 log 等级。与这些 logging 库不同，Go 中 log 包在你调用它的 Fatal*()和 Panic*()函数时，可以做的不仅仅是 log。当你的应用调用这些函数时，Go 也将会终止应用。

```go
package main

import "log"

func main() {
    log.Fatalln("Fatal Level: log entry") //app exits here
    log.Println("Normal Level: log entry")
}
```

#### 22.内建的数据结构操作不是同步的

即使 Go 本身有很多特性来支持并发，并发安全的数据集合并不是其中之一。确保数据集合以原子的方式更新是你的职责。Goroutines 和 channels 是实现这些原子操作的推荐方式，但你也可以使用“sync”包，如果它对你的应用有意义的话。

#### 23.String 在"range"语句中的迭代值

索引值（“range”操作返回的第一个值）是返回的第二个值的当前“字符”（unicode 编码的 point/rune）的第一个 byte 的索引。它不是当前“字符”的索引，这与其他语言不同。注意真实的字符可能会由多个 rune 表示。如果你需要处理字符，确保你使用了“norm”包（golang.org/x/text/unicode/norm）。

string 变量的 for range 语句将会尝试把数据翻译为 UTF8 文本。对于它无法理解的任何 byte 序列，它将返回 0xfffd runes（即 unicode 替换字符），而不是真实的数据。如果你任意（非 UTF8 文本）的数据保存在 string 变量中，确保把它们转换为 byte slice，以得到所有保存的数据。

```go
package main

import "fmt"

func main() {
    data := "A\xfe\x02\xff\x04"
    for _,v := range data {
        fmt.Printf("%#x ",v)
    }
    //prints: 0x41 0xfffd 0x2 0xfffd 0x4 (not ok)

    fmt.Println()
    for _,v := range []byte(data) {
        fmt.Printf("%#x ",v)
    }
    //prints: 0x41 0xfe 0x2 0xff 0x4 (good)
}
```

#### 24.对 Map 使用 for range 语句迭代

如果你希望以某个顺序（比如，按 key 值排序）的方式得到元素，就需要这个技巧。每次的 map 迭代将会生成不同的结果。Go 的 runtime 有心尝试随机化迭代顺序，但并不总会成功，这样你可能得到一些相同的 map 迭代结果。所以如果连续看到 5 个相同的迭代结果，不要惊讶。

```go
package main

import "fmt"

func main() {
    m := map[string]int{"one":1,"two":2,"three":3,"four":4}
    for k,v := range m {
        fmt.Println(k,v)
    }
}
```

#### 25.switch 声明中的失效行为

在“switch”声明语句中的“case”语句块在默认情况下会 break。这和其他语言中的进入下一个“next”代码块的默认行为不同。

```go
package main

import "fmt"

func main() {
	isSpace := func(ch byte) bool {
		switch (ch) {
		case ' ': //error
		case '\t':
			return true
		}
		return false
	}

	fmt.Println(isSpace('\t')) //prints true
	fmt.Println(isSpace(' '))  //prints false
}
```

你可以通过在每个“case”块的结尾使用“fallthrough”，来强制“case”代码块进入。你也可以重写 switch 语句，来使用“case”块中的表达式列表。

```go
package main

import "fmt"

func main() {
	isSpace := func(ch byte) bool {
		switch (ch) {
		case ' ','\t':
			return true
		}
		return false
	}

	fmt.Println(isSpace('\t'))    //prints true
 	fmt.Println(isSpace(' '))     //prints true
}
```

#### 26.自增和自减

许多语言都有自增和自减操作。不像其他语言，Go 不支持前置版本的操作。你也无法在表达式中使用这两个操作符。

Fails:

```go
package main

import "fmt"

func main() {
	data := []int{1,2,3}
	i := 0
	++i //error
	fmt.Println(data[i++]) //error
}
```

Works:

```go
package main

import "fmt"

func main() {
	data := []int{1,2,3}
	i := 0
	i++
	fmt.Println(data[i])
}
```

#### 27.按位 NOT 操作

许多语言使用`~`作为一元的 NOT 操作符（即按位补足），但 Go 为了这个重用了 XOR 操作符（^）。

Fails:

```go
package main

import "fmt"

func main() {
    fmt.Println(~2) //error
}
```

Works:

```go
package main

import "fmt"

func main() {
    var d uint8 = 2
    fmt.Printf("%08b\n",^d)
}
```

Go 依旧使用`^`作为 XOR 的操作符，这可能会让一些人迷惑。

如果你愿意，你可以使用一个二元的 XOR 操作（如， 0x02 XOR 0xff）来表示一个一元的 NOT 操作（如，NOT 0x02）。这可以解释为什么`^`被重用来表示一元的 NOT 操作。

Go 也有特殊的‘AND NOT’按位操作（`&^`），这也让 NOT 操作更加的让人迷惑。这看起来需要特殊的特性/hack 来支持 `A AND (NOT B)`，而无需括号。

```go
package main

import "fmt"

func main() {
    var a uint8 = 0x82
    var b uint8 = 0x02
    fmt.Printf("%08b [A]\n",a)
    fmt.Printf("%08b [B]\n",b)

    fmt.Printf("%08b (NOT B)\n",^b)
    fmt.Printf("%08b ^ %08b = %08b [B XOR 0xff]\n",b,0xff,b ^ 0xff)

    fmt.Printf("%08b ^ %08b = %08b [A XOR B]\n",a,b,a ^ b)
    fmt.Printf("%08b & %08b = %08b [A AND B]\n",a,b,a & b)
    fmt.Printf("%08b &^%08b = %08b [A 'AND NOT' B]\n",a,b,a &^ b)
    fmt.Printf("%08b&(^%08b)= %08b [A AND (NOT B)]\n",a,b,a & (^b))
}
```

#### 28.操作优先级的差异

除了”bit clear“操作（`&^`），Go 也一个与许多其他语言共享的标准操作符的集合。尽管操作优先级并不总是一样。

```go
package main

import "fmt"

func main() {
    fmt.Printf("0x2 & 0x2 + 0x4 -> %#x\n",0x2 & 0x2 + 0x4)
    //prints: 0x2 & 0x2 + 0x4 -> 0x6
    //Go:    (0x2 & 0x2) + 0x4
    //C++:    0x2 & (0x2 + 0x4) -> 0x2

    fmt.Printf("0x2 + 0x2 << 0x1 -> %#x\n",0x2 + 0x2 << 0x1)
    //prints: 0x2 + 0x2 << 0x1 -> 0x6
    //Go:     0x2 + (0x2 << 0x1)
    //C++:   (0x2 + 0x2) << 0x1 -> 0x8

    fmt.Printf("0xf | 0x2 ^ 0x2 -> %#x\n",0xf | 0x2 ^ 0x2)
    //prints: 0xf | 0x2 ^ 0x2 -> 0xd
    //Go:    (0xf | 0x2) ^ 0x2
    //C++:    0xf | (0x2 ^ 0x2) -> 0xf
}
```

#### 29.未导出的结构体不会被编码

以小写字母开头的结构体将不会被（json、xml、gob 等）编码，因此当你编码这些未导出的结构体时，你将会得到零值。

Fails:

```go
package main

import (
	"encoding/json"
	"fmt"
)

type MyData struct {
	One int
	two string
}

func main() {
	in := MyData{1,"two"}
	fmt.Printf("%#v\n",in) // prints main.MyData{One:1,two:"two"}

	encoded,_ := json.Marshal(in)
	fmt.Println(string(encoded)) // prints {"One":1}

	var out MyData
	json.Unmarshal(encoded,&out)
	fmt.Printf("%#v\n",out) // prints main.MyData{One:1,two:""}
}
```

#### 30.有活动的 Goroutines 下的应用退出

应用将不会等待所有的 goroutines 完成。这对于初学者而言是个很常见的错误。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	workerCount := 2

	for i:=0;i<workerCount;i++{
		go doit(i)
	}
	time.Sleep(1 * time.Second)
	fmt.Println("all done!")
}

func doit(workerId int){
	fmt.Printf("[%v] is running\n",workerId)
	time.Sleep(3 * time.Second)
	fmt.Printf("[%v] is done\n",workerId)
}
```

得到如下结果：

```
[1] is running
[0] is running
all done!
```

一个最常见的解决方法是使用“WaitGroup”变量。它将会让主 goroutine 等待所有的 worker goroutine 完成。如果你的应用有长时运行的消息处理循环的 worker，你也将需要一个方法向这些 goroutine 发送信号，让它们退出。你可以给各个 worker 发送一个“kill”消息。另一个选项是关闭一个所有 worker 都接收的 channel。这是一次向所有 goroutine 发送信号的简单方式。

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	done := make(chan struct{})
	workerCount := 2

	for i:=0;i<workerCount;i++{
		wg.Add(1)
		go doit(i,done,wg)
	}

	close(done)
	wg.Wait()
	fmt.Println("all done!")
}

func doit(workerId int,done <-chan struct{},wg sync.WaitGroup){
	fmt.Printf("[%v] is running\n",workerId)
	defer wg.Done()
	<- done
	fmt.Printf("[%v] is done\n",workerId)
}
```

如果你运行这个应用，得到：

```
[0] is running
[0] is done
[1] is running
[1] is done
```

同时会得到:

```
fatal error: all goroutines are asleep - deadlock!
```

死锁发生是因为各个 worker 都得到了原始的“WaitGroup”变量的一个拷贝。当 worker 执行 wg.Done()时，并没有在主 goroutine 上的“WaitGroup”变量上生效。

```go
package main

import (
	"fmt"
	"sync"
)

func main() {
	var wg sync.WaitGroup
	done := make(chan struct{})
	wq := make(chan interface{})
	workerCount := 2

	for i:=0;i<workerCount;i++{
		wg.Add(1)
		go doit(i,wq,done,&wg)
	}

	for i:=0;i < workerCount;i++ {
		wq <- i
	}
	close(done)
	wg.Wait()
	fmt.Println("all done!")
}

func doit(workerId int,wq <- chan interface{},done <- chan struct{},wg *sync.WaitGroup){
	fmt.Printf("[%v] is running\n",workerId)
	defer wg.Done()
	for {
		select {
		case m := <- wq:
			fmt.Printf("[%v] m => %v\n",workerId,m)
		case <- done:
			fmt.Printf("[%v] is done\n",workerId)
			return
		}
	}
}

```

这样既可

#### 31.向无缓存的 Channel 发送信息，只要目标接收者准备好就会立即返回

发送者将不会被阻塞，除非消息正在被接收者处理。根据你运行代码的机器的不同，接收者的 goroutine 可能会或者不会有足够的时间，在发送者继续执行前处理消息。

```go
package main

import "fmt"

func main(){
	ch := make(chan string)

	go func() {
		for m := range ch{
			fmt.Println("processed:",m)
		}
	}()

	ch <- "cmd.1"
	ch <- "cmd.2" //won't be processed
}
```

#### 32.向已关闭的 Channel 发送会引起 Panic

从一个关闭的 channel 接收是安全的。在接收状态下的 `ok` 的返回值将被设置为 `false`，这意味着没有数据被接收。如果你从一个有缓存的 channel 接收，你将会首先得到缓存的数据，一旦它为空，返回的 `ok` 值将变为 `false`。

向关闭的 channel 中发送数据会引起 panic。这个行为有文档说明，但对于新的 Go 开发者的直觉不同，他们可能希望发送行为与接收行为很像。

```go
package main

import (
	"fmt"
	"time"
)

func main(){
	ch := make(chan int)
	for i:=0;i<3;i++{
		go func(idx int) {
			ch <- (idx + 1) * 2
		}(i)
	}

	fmt.Println(<-ch)
	close(ch)
	time.Sleep(2 * time.Second)
}
```

根据不同的应用，修复方法也将不同。可能是很小的代码修改，也可能需要修改应用的设计。无论是哪种方法，你都需要确保你的应用不会向关闭的 channel 中发送数据。

```go
package main

import (
	"fmt"
	"time"
)

func main(){
	ch := make(chan int)
	done := make(chan struct{})
	for i := 0;i < 3; i++{
		go func(idx int) {
			select {
			case ch <- (idx + 1) * 2:
				fmt.Println(idx,"sent result")
			case <- done:
				fmt.Println(idx,"exiting")
			}
		}(i)
	}

	// get first result
	fmt.Println("result:",<-ch)
	close(done)
	// do other work
	time.Sleep(3 * time.Second)
}
```

#### 33.使用 nil Channels

在一个 nil 的 channel 上发送和接收操作会被永久阻塞。

```go
package main

import (
	"fmt"
	"time"
)

func main(){
	var ch chan int
	for i:=0;i<3;i++{
		go func(idx int) {
			ch <- (idx + 1) * 2
		}(i)
	}

	fmt.Println("result:",<-ch)

	time.Sleep(2 * time.Second)
}
```

如果运行代码会看到一个 runtime 错误：

```
fatal error: all goroutines are asleep - deadlock!
```

这个行为可以在 `select` 声明中用于动态开启和关闭 `case` 代码块的方法。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	inch := make(chan int)
	outch := make(chan int)
	go func() {
		var in <- chan int = inch
		var out chan <- int
		var val int
		for {
			select {
			case out <- val:
				out = nil
				in = inch
			case val = <- in:
				out = outch
				in = nil
			}
		}
	}()
	go func() {
		for r := range outch {
			fmt.Println("result:",r)
		}
	}()
	time.Sleep(0)
	inch <- 1
	inch <- 2
	time.Sleep(3 * time.Second)
}
```

#### 34.传值方法的接收者无法修改原有的值

方法的接收者就像常规的函数参数。如果声明为值，那么你的函数/方法得到的是接收者参数的拷贝。这意味着对接收者所做的修改将不会影响原有的值，除非接收者是一个 map 或者 slice 变量，而你更新了集合中的元素，或者你更新的域的接收者是指针。

```go
package main

import "fmt"

type data struct {
	num int
	key *string
	items map[string]bool
}

func (this *data) pmethod() {
	this.num = 7
}

func (this data) vmethod() {
	this.num = 8
	*this.key = "v.key"
	this.items["vmethod"] = true
}

func main() {
	key := "key.1"
	d := data{1,&key,make(map[string]bool)}
	fmt.Printf("num=%v key=%v items=%v\n",d.num,*d.key,d.items)
	//prints num=1 key=key.1 items=map[]
	d.pmethod()
	fmt.Printf("num=%v key=%v items=%v\n",d.num,*d.key,d.items)
	//prints num=7 key=key.1 items=map[]
	d.vmethod()
	fmt.Printf("num=%v key=%v items=%v\n",d.num,*d.key,d.items)
	//prints num=7 key=v.key items=map[vmethod:true]
}
```
