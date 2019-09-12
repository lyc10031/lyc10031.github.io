---
layout: article
title: Go 的slice和array
tags: 
- GO
- slice
- array
- 学习记录
key: go-tips
aside:
  toc: true
---

## Array

```
func main() {
    nums := [3]int{} //定义并初始化一个array
    nums[0] = 1

    n := nums[0]
    n = 2

    fmt.Printf("nums: %v\n", nums)
    fmt.Printf("n: %d\n", n)
}
```

我们可得知在 Go 中，数组类型需要指定长度和元素类型。在上述代码中，可得知 [3]int{} 表示 3 个整数的数组，并进行了初始化。底层数据存储为一段连续的内存空间，通过固定的索引值（下标）进行检索

- 数组类型需要指定长度和元素类型。
- 数组在声明后，其元素的初始值（也就是零值）为 0。并且该变量可以直接使用，不需要特殊操作。
- 同时数组的长度是固定的，它的长度是类型的一部分。因此 [3]int 和 [4]int 在类型上是不同的，不能称为 “一个东西”。

## slice

slice 本质上是在数组之上提供了访问指定范围内（起始索引和终止索引）的数组数据。

```
func main() {
    nums := [3]int{}
    nums[0] = 1

    dnums := nums[:]

    fmt.Printf("dnums: %v", dnums)
}
```
## golang 的堆和栈


```
var p *int    //全局指针变量
func f(){
    var i int
    i = 1
    p = &i    //全局指针变量指向局部变量i
}
```

```
func f(){
    p := new(int) //局部指针变量，使用new申请的空间
    *p = 1
}
```
- 上面程序中，第一个程序虽然i是通过var申请的局部变量，但是由于有外部指针指向访问，我们有路径可找到这个空间（变量能够逃逸出函数），所以局部变量i是申请在堆空间上。而第二个程序中p指针变量虽然是使用new申请的空间，但是由于退出函数就没有路径可寻找到它（变量无法逃出函数），所以局部变量p是申请在栈空间上的。
