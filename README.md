Go 语言中常见的学习错误，根据 Go 的 50 度灰整理而得。
原文：[50 Shades of Go: Traps, Gotchas, and Common Mistakes for New Golang Devs](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)

<!--more-->

[toc]

### 1.初级

#### 1.1 开大括号不能放在单独的一行

```go
// 错误示例
package main

import "fmt"

func main()
{   //error, can't have the opening brace on a separate line
    fmt.Println("Hello World!")
}
```

#### 1.2 未使用的变量

```go

```
