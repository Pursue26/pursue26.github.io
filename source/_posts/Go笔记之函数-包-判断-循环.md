---
title: Go笔记之函数&包&判断&循环
date: 2023-09-09 12:01:45
categories: 
    - 学习笔记
    - Golang
tags:
    - Go函数
    - Go判断
    - Go循环
abbrlink: 230909120145
---

## Go语言学习笔记-5：函数

1. 函数声明语法：

```go
func functionname(parametername1 type, parametername2 type) returntype {  
    // 函数体（具体实现的功能）
}
```

2. 如果有连续若干个参数，它们的类型一致，那么我们无须一一罗列，只需在最后一个参数后添加该类型。

```go
func functionname(parametername1, parametername2 type) returntype {  
    // 函数体（具体实现的功能）
}
```

3. Go 语言支持一个函数可以有**多个返回值**。

```go
func functionname(para1 type1, para2 type2) (returntype1, returntype2, ...) {
    // 函数体（具体实现的功能）
}
```
<!-- more -->

4. **命名返回值**：从函数中可以返回一个命名值。一旦命名了返回值，可以认为**这些值在函数第一行就被声明为变量了**，在函数 return 时不必再跟随命名值。如`area, perimeter`：

```go
func rectProps(length, width float64) (area, perimeter float64) {  
    area = length * width
    perimeter = (length + width) * 2
    return // 不需要明确指定返回值，默认返回 area, perimeter 的值
}
```

5. **空白符**：`_` 在 Go 中被用作空白符，可以用作表示任何类型的任何值。

`area, _ := rectProps(10.8, 5.6) // 返回值周长被丢弃`。

---

## Go语言学习笔记-6：包

1. `package packagename` 指定了某一源文件属于一个包，它应该放在每一个源文件的第一行。

2. `main` 包和自定义包目录结构（属于某一个包的源文件都应该放置于一个单独命名的文件夹里，且应该用包名命名文件夹名）：

```go
src
    geometry  // 自定义 main 包
        geometry.go // main 函数
        rectangle // 自定义包，文件名为包名
            rectprops.go // 属于 rectangle 包的源文件
bin
    // 通过执行 go build geometry/geometry.go 编译得到 
    geometry (Linux)
    geometry.exe (Windows)
```

3. 导入自定义包：`import packagepath`，必须指定自定义包 `packagename` 相对于工作区 `src` 文件夹的相对路径，如：`import "geometry/rectangle"`。

4. 导出名字：在 Go 中，任何**以大写字母开头的变量或者函数都是被导出的名字，其它包只能访问被导出的函数和变量**（即，如果想在包外访问一个函数，它应该首首字母大写）。

5. `init` 函数：所有包都可以包含一个 `init` 函数，`init` 函数不应该有任何返回值类型和参数。

- 包的初始化顺序：1）首先初始化**被导入的**包；2）然后初始化**包级别（package level）的变量**；3）紧接着**调用 `init` 函数**，按照编译器解析它们的顺序进行调用。

- 如果一个包导入了另一个包，会先初始化**被导入的**包。

- 尽管一个包可能会被导入多次，但是它们只会被初始化一次。

6. 空白标识符导入包中的使用：导入了包，却不在代码中使用它，这在 Go 中是非法的，会抛出 `xxx.go:6: imported and not used: packagename_yyy`。

- 为了避免这种程序错误，通常会在函数外调用其中的一个函数，并将返回值赋给 `_` 变量，把这一操作称为`错误屏蔽器`。

- 同时，有时我们并不想使用导入的包中的任一函数或变量，只是为了**确保它进行初始化**，这种情况可以使用空白标识符 `_`，如 `import _ "geometry/rectangle"`。这样在不调用包中的函数或变量时，也不会报错。

---

## Go语言学习笔记-7：if-else

1. 即使 `if` 状态下仅有一条语句，也必须加 `{ }`

```go
if condition {

} else if condition {

} else {

}
```

2. `if` 还有另外一种形式，它包含一个 `statement` 可选语句部分，该组件在条件判断之前运行，语法为：`if statement; condition {  }`，其中 `statement` 的作用域仅在 `if-else` 范围内。

```go
func main() { // num 的范围仅限于 if else 代码块
  if num:= 10; num % 2 == 0 {
    fmt.Println(num, "is even.")
  } else {
    fmt.Println(num, "is odd.")
  }
}
```

3. **一个注意点**：`else` 必须键入在 `}` 后面，不能另取一行键入，因为 Go 语言默认在每条语句结束时插入一个分号，但 `if else` 是一个整体。

## Go语言学习笔记-8：循环

1. Go 语言中唯一的循环语句是 `for`，没有 `while` 和 `do while` 循环。

```go
for initialisation; condition; post {  
    // 循环体
}
```

- `for` 循环的三个组成部分，即初始化，条件和 post 都是可选的（可有可无），如：`for ;i <= 10; {...}`、`for {...}` 或 `for i <= 10 {... i += 2}`，即 `;` 也是可以省略的。

2. `break` 为跳出循环，`continue` 为不执行后续语句，进入下一次循环。

---

## Go语言学习笔记-9：switch语句

1. `switch`是一个条件语句，它可以被认为是替代多个 `if else` 子句的常用方式，`case` 不允许出现重复项。

```go
func main() {
    finger := 4
    switch finger {
        case 1:
            fmt.Println("Thumb")
        case 2:
            fmt.Println("Index")
        case 3:
            fmt.Println("Middle")
        case 4, 5: // 一个选项多个表达式
            fmt.Println("Ring or Pinky")
        default: // 默认情况
            fmt.Println("incorrect finger number")
    }
}
```

2. **先声明变量再使用**：`switch varname := xxx; varname {...}`，此时的 `varname` 变量的作用域仅限于当前 `switch` 内。

3. **无表达式的 `switch`**：在 switch 语句中，表达式是可选的，可以被省略。如果省略表达式，则表示这个 switch 语句**等同于 switch true**，并且**每个 case 表达式都被认定为有效，相应的代码块也会被执行**。

4. **fallthrough**：fallthrough 语句可以在已经执行完成的 case 之后，把控制权转移到下一个 case 的执行代码中，而不会跳出 switch。

```go
func  main() {
    switch num := 3; {
        case num == 3:
            fmt.Println("==")
            fallthrough
        case num < 10:
            fmt.Println("yes")
    }
}
```

5. switch 和 case 的表达式不一定是常量。

> 学习链接：[https://studygolang.com/subject/2](https://studygolang.com/subject/2)，感谢如此优秀的教程！
