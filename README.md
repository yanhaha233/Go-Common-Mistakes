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
