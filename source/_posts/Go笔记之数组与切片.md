---
title: Go笔记之数组与切片
date: 2023-09-12 15:28:00
categories: 
    - 学习笔记
    - Golang
tags:
    - Go数组
    - Go切片
abbrlink: 230912152800
---

## Go语言学习笔记-10：数组

1. 数组是**同一类型元素**的集合，Go语言中不允许混合不同类型的元素。

2. 一个数组的表示形式为 `[n]T`。n 表示数组中元素的数量，T 代表每个元素的类型。如： `var arr [3]int`。

3. 数组简略声明：`arr := [3]int{12, 78, 50}`，在简略声明中，可对部分元素赋值：`arr := [3]int{12}`，剩下的元素自动赋值为 0 。

4. 可以忽略声明数组的长度，并用 `...` 代替，让编译器为你自动计算长度：`arr := [...]int{12, 21}`。

5. **数组是值类型，不是引用类型**：这意味着当数组赋值给一个新的变量时，该变量会得到一个原始数组的副本。**如果对新变量进行更改，则不会影响原始数组**。注：对切片的修改会影响原始数组。

<!--more-->

6. **当数组作为参数传递给函数时，它们是按值传递，而原始数组保持不变（原始数组不会因为函数调用而改变）**。

```go
func changeLocal(num [5]int) {
    num[0] = 55
    fmt.Println("inside function ", num)
}

func main() {
    num := [...]int{5, 6, 7, 8, 8}
    fmt.Println("before passing to function ", num)
    changeLocal(num) // num is passed by value
    fmt.Println("after passing to function ", num)
}

// before passing to function  [5 6 7 8 8]
// inside function  [55 6 7 8 8]
// after passing to function  [5 6 7 8 8]
```

7. 数组的长度：`len(arr)`。

8. 数组的迭代 for 循环：

```go
for i := 0; i < len(arr); i++ {
    fmt.Println("%d-th element of arr is %.2f\n", i, arr[i])
}
```

9. 数组的迭代：Go 提供了一种更好、更简洁的方法，通过**使用 for 循环的 `range` 方法来遍历数组**。`range` 返回索引和该索引处的值。

```go
for i, v := range arr {
  fmt.Println("%d-th element of arr is %.2f\n", i, v)
}
```

10. 多维数组：1）简略声明；2）先声明二维数组变量，再根据索引来对数组添加值。

```go
arr := [3][2]string{ // 简略声明
      {"lion", "tiger"},
      {"cat", "dog"},
      {"pigeon", "peacock"}, // this comma is necessary. The compiler will complain if you omit this comma
}

for _, v1 := range arr { // 打印二维数组
    for _, v2 := range v1 {
        fmt.Printf("%s ", v2)
    }
    fmt.Printf("\n")
}
```

11. 数组的缺陷：长度固定，不可能增加数组的长度。

---

## Go语言学习笔记-11：切片

1. 切片是**由数组建立**的一种方便、灵活且功能强大的包装（Wrapper）。**切片本身不拥有任何数据**。它们只是对现有数组的**引用**。

2. 创建一个切片 A ：带有 `T` 类型元素的切片由 `[]T` 表示。语法：`a[start:end]` 创建一个从 a 数组索引 start 开始到 end - 1 结束的切片。

```go
arr := [5]int{76, 77, 78, 79, 80}
var b []int = a[1:4] // 数组切片
```

3. 创建一个切片 B ：创建一个数组，并返回一个存储在 c 中的**切片引用**。注意：`[]` 内无内容。

```go
c := []int{6, 7, 8} // creates and array and returns a slice reference
```

4. **对切片的修改会影响原始数组**。

5. 当多个切片共用相同的底层数组时，每个切片所做的更改将反映在数组中。

6. **切片的长度和容量**：切片的长度是切片中的元素数。切片的容量是从创建切片索引开始（到数组末尾）的底层数组中的元素数。

```go
func main() {
    fruitarray := [...]string{"a", "b", "c", "d"}
    fruitslice := fruitarray[1:3]
    fmt.Println("length of slice: %d, capacity: %d", len(fruitslice), cap(fruitslice)) // 2, 3
}
```

7. 切片可以重置其容量：`slicename[:cap(slicename)]`。

```go
func main() {
    fruitarray := [...]string{"a", "b", "c", "d"}
    fruitslice := fruitarray[1:3]
    fmt.Println("length of slice: %d, capacity: %d", len(fruitslice), cap(fruitslice)) // 2, 3. print: [b, c]
    fruitslice = fruitslice[:cap(fruitslice)] // re-slicing furitslice till its capacity. print: [b, c, d]
    fmt.Println("After re-slicing length is", len(fruitslice), "and capacity is", cap(fruitslice)) // 3, 3
}
```

8. **使用 `make` 创建一个切片**：`func make ([]T, len, cap)` 通过传递类型、长度和容量来创建切片。其中，容量是可选参数，默认值为切片长度。**`make` 函数创建一个数组，并返回引用该数组的切片**。如 `arrslice := make([]float64, 5, 5)`。

9. 切片是动态的，可以使用 `append` 将新元素追加到切片上。`append` 函数定义为：`func append(slice []T, x ...T) []T`，其中 `x...T` 表示该函数接收 T 类型的参数 x 的个数是可变的，如：`arrslice = append(arrslice, 5.6, 7.3)`。

> `append` 函数会返回一个新的切片，其中包含了原始切片和追加的元素：
> 1.    如果原始切片的容量足够，`append` 函数会在原始切片的基础上进行追加；
> 2.    如果原始切片的容量不够，`append` 函数会创建一个新的底层数组，并将原始切片中的元素和追加的元素复制到新的底层数组中。
> 
> 因此，append 函数返回的切片可能指向一个新的底层数组，而不是原始切片所指向的底层数组。

10. 数组是固定的，但**切片具有动态长度**：切片是由一个指向数组的指针、长度和容量组成的数据结构。当我们向切片中追加元素时，如果切片的长度小于容量，新元素会直接添加到切片的末尾，切片的长度会增加。但是，如果追加元素后切片的长度超过了容量，Go 语言会创建一个新的更大的底层数组，将原来的元素复制到新的数组中，并将新元素添加到新数组的末尾。然后，切片会指向这个新数组（**相比于旧切片，引用类型的地址改变了**），并且容量会成为原来的两倍（并不是每次执行 `append` 时容量都会变成旧切片的两倍，只有当长度超过容量时，才会扩充一倍容量）。这样，切片就具有了动态长度的特性。

11. 切片类型的零值为 `nil`：一个 `nil` 切片的长度和容量为 0，可以使用 `append` 函数将值追加到 `nil` 切片。

```go
var names []string // zero value of a slice is nil
if names == nil {
    fmt.Println("slice is nil going to append")
    names = append(names, "John", "Sebastian", "Vinay")
    fmt.Println("names contents:", names)
}
```

12. `...` 可将一个切片添加到另一个切片中：`append(slice1, slice2...)`。

13. 切片的函数传递：切片在内部可由一个结构体类型表示，即：

```go
type slice struct {
    Length         int
    Capacity       int
    ZerothElement  *byte
}
```

-   切片本身包含了长度、容量和指向底层数组首个元素的指针。当切片作为参数传递给函数时，虽然是通过值传递，但是切片内部的指针变量（这里的`byte`）仍然指向相同的底层数组。因此，当函数内部修改底层数组的值时，这些修改在函数外部是可见的。

-   然而，如果函数内部修改了切片的长度或容量，将会创建一个新的切片，而不会影响原始切片。这是因为切片的长度和容量是切片结构体的字段，而非底层数组的属性。因此，修改切片的长度或容量会创建一个新的切片结构体，其中的指针变量仍然指向原始的底层数组，但是新的切片具有不同的长度和容量。

-   因此，可以说当切片传递给函数时，函数内部对底层数组的修改，在函数外部是可见的，但是对切片的长度和容量的修改是不可见的。

```go
func modifySlice(s []int) {
    s[0] = 100 // 修改底层数组的值
    s = append(s, 200) // 创建新的切片，不影响原始切片
    fmt.Println("Inside modifySlice:", s)
}

func main() {
    numbers := []int{1, 2, 3, 4, 5}
    fmt.Println("Before modifySlice:", numbers)
    modifySlice(numbers)
    fmt.Println("After modifySlice:", numbers)
}

// Before modifySlice: [1 2 3 4 5]
// Inside modifySlice: [100 2 3 4 5 200]
// After modifySlice: [100 2 3 4 5]  // 函数内部对底层数组的修改在函数外部是可见的
```

14. **多维切片**：类似于多维数组，但每行索引对应的元素数量可以不等于其它行。

```go 
pls := [][]string {
      {"C", "C++"},
      {"JavaScript"},
      {"Go", "Rust"},
}
```

15. 内存优化：切片持有对底层数组的引用，只要切片在内存中，数组就不能被垃圾回收。一种回收数组的方法时使用 `copy` 函数 `func copy(dst, src[]T) int` 来生成一个切片的副本，这样就可以使用新的切片，原始数组也可以被垃圾回收。注：返回的 int 类型的值为 dst 的长度。

```go
func countries() []string {
    countries := []string{"USA", "Singapore", "Germany", "India", "Australia"}
    neededCountries := countries[:len(countries)-2]
    countriesCpy := make([]string, len(neededCountries))
    copy(countriesCpy, neededCountries) //copies neededCountries to countriesCpy
    return countriesCpy
}
func main() {
    countriesNeeded := countries()
    fmt.Println(countriesNeeded)
}
```

> 学习链接：[https://studygolang.com/subject/2](https://studygolang.com/subject/2)，感谢如此优秀的教程！
