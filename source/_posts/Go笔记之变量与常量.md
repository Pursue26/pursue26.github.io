---
title: Go笔记之变量与常量
date: 2023-09-07 16:26:12
categories: 
    - 学习笔记
    - Golang
tags:
    - Go变量与常量
abbrlink: 230907162612
---

## Go语言学习笔记-1：HelloWorld

Go语言是一种编译型语言，源代码都会编译成二进制机器码。

所有 Go 源文件都应该放置在工作区里的 src 目录下。Linux 的工作区（Workspace）应该设置在 $HOME/go，也可以通过设置 GOPATH 环境变量，用其他目录来作为工作区。

```go
go
  bin // 编译生成的二进制文件存储位置
    hello
  src // 所有 Go 源文件都应该放置在工作区里的 src 目录下
    hello // 为每个project新建一个文件夹
      helloworld.go
```

第一个Go程序

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello World!")
}
```

<!-- more -->

解析：
- `package main` - 每一个 Go 文件都应该在开头进行 package name 的声明（译注：只有可执行程序的包名应当为 main）。包（Packages）用于代码的封装与重用，这里的包名称是main。

- `import "fmt"` - 我们引入了 fmt 包，用于在 main 函数里面打印文本到标准输出。

- `func main()` - main 是一个特殊的函数。整个程序就是从 main 函数开始运行的。main 函数必须放置在 main 包中。{ 和 } 分别表示 main 函数的开始和结束部分。

- `fmt.Println("Hello World")` - fmt 包中的 Println 函数用于把文本写入标准输出。

---

## Go语言学习笔记-2：变量

**变量**指定了某存储单元（Memory Location）的名称，该存储单元会存储特定类型的值。

1. 声明单个变量：var name type，例如：var age int

2. 声明变量并初始化：var name type = initialValue

3. 类型推断：如果变量有初始值，那么 Go 能够自动推断具有初始值的变量的类型。因此，如果变量有初始值，就可以在变量声明中省略 type， 如 var age = 16 指 age 类型为 int 型。

4. 声明多个变量：var name1, name2 type = initialValue1, initialValue2

5. 在一个语句中声明不同类型的变量，如：

```go
var (
    age = 19
    name = “laowang”
    adult = true
)
```

6. 变量的简短声明语法：`:=` 操作符，例如：`age, name := 20, "zhangsan"`。简短声明要求 `:=` 操作符左边的**所有变量都有初始值**。

```go
func main() {  
    a, b := 20, 30 // 声明a和b
    fmt.Println("a is", a, "b is", b)
    a, b := 40, 50 // 错误，没有尚未声明的变量
    a, c := 40, 60 // 正确，有新的尚未声明的变量
}
```

7. 变量也可以在运行时进行赋值，如 `a := math.Min(15, 20)`

8. 由于 Go 是强类型（Strongly Typed）语言，因此不允许某一类型的变量赋值为其他类型的值。

```go
func main() {  
    age := 29      // age是int类型
    age = "naveen" // 错误，尝试赋值一个字符串给int类型变量
}
```

---

## Go语言学习笔记-3：变量类型

1. Go语言支持的变量类型：

```text
bool
数字类型
    int8, int16, int32, int64, int
    uint8, uint16, uint32, uint64, uint
    float32, float64
    complex64, complex128 // 复数型
    byte
    rune
string
```

2. 在 Printf 方法中，使用 `%T` 格式说明符（Format Specifier），可以打印出变量的类型。Go 的 `unsafe` 包提供了一个 `Sizeof` 函数，该函数接收变量并返回它的字节大小。如：

```go
fmt.Printf("type of age is %T, size of age is %d", age, unsafe.Sizeof(age))
```

3. 复数类型：`complex64` 表示实部和虚部都是 `float32` 类型，`complex128` 表示实部和虚部都是 `float64` 类型。

```go
c1 := complex(5, 7) // 通过内置函数声明复数变量c1
c2 := 8 + 27i // 通过简短声明声明复数变量c2
```

4. 其它变量类型：`byte` 是 `uint8` 的别名， `rune` 是 `int32` 的别名。

5. Go 是强类型（Strongly Typed）语言， Go 没有自动类型提升或类型转换：

```go
var i, j int, float = 5, 10.2
diff := i - j // 错误， int - float 不被允许
diff := i - int(j) // 正确
```

---

## Go语言学习笔记-4：常量

1. 关键字：`const`，常量不能再重新赋值为其他的值。

2. 常量的值会在**编译的时候**确定。因为函数调用发生在**运行时**，所以不能将函数的返回值赋值给常量。如：

```go
var a := math.Sqrt(4) // 允许
const b := math.Sqrt(9) //不允许，编译先于运行
```

3. 常量是可以没有类型的，如：`const name = "zhangsan"`；常量也可以是有类型的，如：`const name string = "lisi"`，即 name 是一个 string 类型的**常量**。

4. Go 的类型策略不允许将一种类型的变量赋值给另一种类型的变量。

```go
func main() {  
    var defaultName = "Sam" // 允许
    type myString string
    var customName myString = "Sam" // 允许
    customName = defaultName // 不允许，即使 myString 是 string 类型的别名
}
```

5. **布尔常量**：字符串常量的规则适用于布尔常量。

`var name = "Sam"` 无类型的**常量** Sam 是如何赋值给**变量** name 的？

> 答案是**无类型的常量有一个与它们相关联的默认类型，并且当且仅当一行代码需要时才提供它**（常量可以赋值给 “合适的” 类型，而不需要类型转换）。在声明变量时，如果使用常量来赋值，变量会从常量的默认类型中获取类型。在这种情况下，常量 "Sam" 的默认类型是字符串，所以变量 name 的类型也是字符串。

> 学习链接：[https://studygolang.com/subject/2](https://studygolang.com/subject/2)，感谢如此优秀的教程！
