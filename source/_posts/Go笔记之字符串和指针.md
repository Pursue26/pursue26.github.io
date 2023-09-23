---
title: Go笔记之字符串和指针
date: 2023-09-15 15:51:05
categories: 
    - 学习笔记
    - Golang
tags:
    - Go字符串
    - Go指针
abbrlink: 230915155105
---

## Go语言学习笔记-14：字符串

1. Go 语言中的字符串是一个**字节切片**。

2. `%x` 格式限定符用于指定 16 进制编码，`%c` 格式限定符用于打印字符串的字符。

3. 在 UTF-8 编码中，一个代码点（字符的编码）可能会占用超过一个字节的空间，所以使用 `%c` 格式打印时可能会出错，可以用 `rune` 解决。

4. `rune` 是 Go 语言的内建类型，它也是 `int32` 的别称。在 Go 语言中，`rune` 表示一个代码点。代码点无论占用多少个字节，都可以用一个 `rune` 来表示。

<!--more-->

```go
s := "Señor"
for i:= 0; i < len(s); i++ {
    fmt.Printf("%c ",s[i]) // 打印出错，S e Ã ± o r
}
fmt.Printf("\n")

runes := []rune(s)
for i:= 0; i < len(runes); i++ { // 字符串被转化为一个 rune 切片
    fmt.Printf("%c ",runes[i]) // 打印正确，S e ñ o r
}
```

5. 字符串的 for range 循环更简单。

```go
func printCharsAndBytes(s string) {
    for index, rune := range s {
        fmt.Printf("%c starts at byte %d\n", rune, index)
    }
}

func main() {
    name := "Señor"
    printCharsAndBytes(name)
}
/*
S starts at byte 0  
e starts at byte 1  
ñ starts at byte 2  // ñ 占了两个字节
o starts at byte 4  
r starts at byte 5
*/
```

6. 用字节切片构造字符串：

```go
// 包含字符串 Café 用 UTF-8 编码后的 16 进制字节
byteSlice := []byte{0x43, 0x61, 0x66, 0xC3, 0xA9}
str := string(byteSlice)
fmt.Println(str) // Café
```

7. 用 `rune` 切片构造字符串：

```go
// 包含字符串 Señor 的 16 进制的 Unicode 代码点
runeSlice := []rune{0x0053, 0x0065, 0x00f1, 0x006f, 0x0072}
str := string(runeSlice)
fmt.Println(str) // Señor
```

8. 字符串的长度：utf-8 package 包中的 `func RuneCountInString(s string) (n int)` 方法用来**获取字符串的长度**。这个方法传入一个字符串参数然后**返回字符串中的 rune 的数量**。

```go
word1 := "Señor" 
fmt.Println(utf8.RuneCountInString(word1)) // 5
word2 := "Pets"
fmt.Println(utf8.RuneCountInString(word2)) // 4
```

9. 字符串是不可变的：一旦一个字符串被创建，它将不可被修改。为了修改字符串，可以把字符串转化为一个 rune 切片，这个切片可以进行任何想要的改变，最后再转化为一个字符串。

```go
func mutate(s []rune) string {  
    s[0] = 'a' // 修改切片
    return string(s) // 转换为一个字符串
}

func main() {  
    h := "hello"
    fmt.Println(mutate([]rune(h))) // 用 rune 切片构造字符串，输出 aello
    fmt.Println(h) // 输出 hello
}
```

---

## Go语言学习笔记-15：指针

1. 指针是一种存储变量内存地址（Memory Address）的变量。例如，变量 b 的值为 156，而 b 的内存地址为 0x1040a124，变量 a 存储了 b 的地址。我们就称 a 指向了 b。

2. 指针的声明：指针变量的类型为 `*T`，该指针指向一个 `T` 类型的变量。

-   & 操作符用于获取变量的地址。下面的程序把 b 的地址赋值给 `*int` 类型的 a。我们称 a 指向了 b。当我们打印 a 的值时，会打印出 b 的地址。

```go
func main() {
    b := 255
    var a *int = &b
    fmt.Printf("Type of a is %T\n", a) // *int
    fmt.Println("address of b is", a)  // 0x1040a124
}
```

3. 指针的零值（Zero Value）是 `nil`。声明变量 `var b *int`，此时 `b == nil` 为 `true`。

4. 指针的解引用：指针的解引用可以获取指针所指向的变量的值。将 `a` 解引用的语法是 `*a`。

```go
func main() {  
    b := 255
    a := &b
    fmt.Println("address of b is", a) // 0x1040a124
    fmt.Println("value of b is", *a)  // 255
    *a++  // (*a)++
    fmt.Println("new value of b is", b) // 256
}
```

5. 向函数传递指针参数：

```go
func change(val *int) {  
    *val = 55
}

func main() {  
    a := 58
    fmt.Println("value of a before function call is", a) // 58
    b := &a
    change(b)
    fmt.Println("value of a after function call is", a) // 55
}
```

6. 假如我们想要在函数内修改一个数组，并希望调用函数的地方也能得到修改后的数组，一种解决方案是把一个指向数组的指针传递给这个函数（但Go语言习惯的方法是用切片处理，见序号7）。

```go
func modify(arr *[3]int) {  
    (*arr)[0] = 90
}

func main() {  
    a := [3]int{89, 90, 91}
    modify(&a)
    fmt.Println(a)
}
```

- `a[x]` 是 `(*a)[x]` 的简写形式，因此上面代码中的 `(*arr)[0]` 可以替换为 `arr[0]`。

7. **不要向函数传递数组的指针，而应该使用数组切片**：

```go
func modify(sls []int) {  
    sls[0] = 90
}

func main() {  
    a := [3]int{89, 90, 91}
    modify(a[:]) // 使用切片
    fmt.Println(a)
}
```
- 别再传递数组指针了，而是使用切片吧。上面的代码更加简洁，也更符合 Go 语言的习惯。

> 学习链接：[https://studygolang.com/subject/2](https://studygolang.com/subject/2)，感谢如此优秀的教程！
