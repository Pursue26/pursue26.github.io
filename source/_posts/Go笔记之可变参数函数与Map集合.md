---
title: Go笔记之可变参数函数与Map集合
date: 2023-09-13 16:13:26
categories: 
    - 学习笔记
    - Golang
tags:
    - Go可变参数函数
    - Go集合Map
abbrlink: 230913161326
---

## Go语言学习笔记-12：可变参数函数

1. 可变参数函数是一种**参数个数可变**的函数。如果函数最后一个参数的类型被记作 `...T` ，这时函数可以接受任意个 `T` 类型的参数作为最后一个参数。**只有**函数的最后一个参数才允许是可变的。

2. 可变参数函数的**工作原理**是把可变参数转换为一个新的**切片**。

3. 有一个可以直接将切片传入可变参数函数的语法糖，你可以在切片后加上 `...` 后缀。这样切片将直接传入函数，不再创建新的切片。

<!--more-->

```go
func find(num int, nums ...int) {
    // ...
}

func main() {
    find(89, 88, 89, 90) // 语法正确，[88, 89, 90] 将被转换为一个新的切片传入 find 函数
    find(89) // 语法正确, 一个长度和容量为 0 的 nil 切片将被传入 find 函数
    nums := []int{87, 88, 89}
    find(89, nums) // 语法错误，find 的可变参数要求为 int 型，不能传入 []int 切片
    find(89, nums...) // 语法正确，切片直接传入函数，不再创建新的切片
}
```

## Go语言学习笔记-13：Maps

1. map 是在 Go 中将值（value）与键（key）关联的**内置类型**。通过相应的键可以获取到值。

2. 通过向 `make` 函数传入键和值的类型，可以创建 map。`make(map[type of key]type of value)` 是创建 map 的语法。如：`personSalary := make(map[string]int)`。

3. map 的零值是 `nil`。如果你想添加元素到 nil map 中，会触发运行时 `panic`。因此 **map 必须使用 `make` 函数初始化**。

```go
func main() {  
    var personSalary map[string]int

    if personSalary == nil {
        fmt.Println("map is nil. Going to make one.")
        personSalary = make(map[string]int) // 使用 make 方法初始化
    }

    personSalary["steve"] = 12000 // 给 map 添加新元素
    personSalary["jamie"] = 15000
    personSalary["mike"] = 9000
    fmt.Println("personSalary map contents:", personSalary) // map[steve:12000 jamie:15000 mike:9000]
}
```

4. 在声明时初始化 map：

```go
personSalary := map[string]int {
        "steve": 12000,
        "jamie": 15000, // 逗号不可缺
}
```

5. 获取 map 中的元素：`map[key]`。如果获取一个不存在的元素，map 会返回**该元素类型的零值**（如：`[]T` 返回 `nil`，整形返回 `0`）。

6. 如何判断某个 key 是否存在于 map 中：`value, ok := map[key]`，如果 ok 是 true，表示 key 存在，key 对应的值就是 value ，反之表示 key 不存在。

7. 遍历 map 中所有的元素需要用 for range 循环：`for key, value := range personSalary {//... }`。当使用 for range 遍历 map 时，不保证每次执行程序获取的元素顺序相同。

8. 删除 map 中的元素：`delete(map, key)`，无返回值。

9. 获取 map 的长度：`len(map)`。

10. 和 slices 类似，map 也是**引用类型**：当 map 被赋值为一个新变量的时候，它们指向同一个内部数据结构。因此，**改变其中一个变量，就会影响到另一变量。当 map 作为函数参数传递时也会发生同样的情况，函数中对 map 的任何修改，对于外部的调用都是可见的**。

11. map 之间不能使用 `==` 操作符判断，`==` 只能用来检查 map 是否为 nil。

> 学习链接：[https://studygolang.com/subject/2](https://studygolang.com/subject/2)，感谢如此优秀的教程！
