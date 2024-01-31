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

- slice 的数据结构：
```
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```
    - array：指向所引用的数组指针（***unsafe.Pointer 可以表示任何可寻址的值的指针***）
    - len：长度，当前引用切片的元素个数
    - cap：容量，当前引用切片的容量（底层数组的元素总数
- **在实际使用中，cap 一定是大于或等于 len 的。否则会导致 panic**

- Slice（切片）是抽象在 Array（数组）之上的特殊类型。
- slice 本质上是在数组之上提供了访问指定范围内（起始索引和终止索引）的数组数据。


```
func main() {
    nums := [3]int{}
    nums[0] = 1

    dnums := nums[:]

    fmt.Printf("dnums: %v", dnums)
}
```
- dnums 变量通过 nums[:] 进行赋值。需要注意的是，Slice 和 Array 不一样，它不需要指定长度。也更加的灵活，能够自动扩容。
- cap扩容计算策略: 小于 1024 个时，增长 2 倍。大于 1024 个时，增长 1.25 倍.

### slice 的两个注意点

1. 同根

    ```
    func main() {
        nums := [3]int{}
        nums[0] = 1

        fmt.Printf("nums: %v , len: %d, cap: %d\n", nums, len(nums), cap(nums))

        dnums := nums[0:2]
        dnums[0] = 5

        fmt.Printf("nums: %v ,len: %d, cap: %d\n", nums, len(nums), cap(nums))
        fmt.Printf("dnums: %v, len: %d, cap: %d\n", dnums, len(dnums), cap(dnums))
    }
    ```

    - 结果：
        ```
        nums: [1 0 0] , len: 3, cap: 3
        nums: [5 0 0] ,len: 3, cap: 3
        dnums: [5 0], len: 2, cap: 3
        ```

    - 在未扩容前，Slice array 指向所引用的 Array。因此在 Slice 上的变更。会直接修改到原始 Array 上（两者所引用的是同一个）。

2. 扩容后修改元素值

    ```
    func main() {
        nums := [3]int{}
        nums[0] = 1

        fmt.Printf("nums: %v , len: %d, cap: %d\n", nums, len(nums), cap(nums))

        dnums := nums[0:2]
        dnums = append(dnums, []int{2, 3}...)
        dnums[1] = 1

        fmt.Printf("nums: %v ,len: %d, cap: %d\n", nums, len(nums), cap(nums))
        fmt.Printf("dnums: %v, len: %d, cap: %d\n", dnums, len(dnums), cap(dnums))
    ```
    - 往 Slice append 元素时，若满足扩容策略，也就是假设插入后，原本数组的容量就超过最大值了。

    - 这时候内部就会重新申请一块内存空间，将原本的元素拷贝一份到新的内存空间上。此时其与原本的数组就没有任何关联关系了，再进行修改值也不会变动到原始数组。这是需要注意的。

### copy
- copy 函数将数据从源 Slice复制到目标 Slice。它返回复制的元素数。
- copy 函数支持在不同长度的 Slice 之间进行复制，若出现长度不一致，在复制时会按照最少的 Slice 元素个数进行复制。
    - 若源 Slice 或目标 Slice 存在长度为 0 的情况，则直接返回 0（因为压根不需要执行复制行为）
    - 通过对比两个 Slice，获取最小的 Slice 长度。便于后续操作
    - 若 Slice 只有一个元素，则直接利用指针的特性进行转换
    - 若 Slice 大于一个元素，则从 fm.array 复制 size 个字节到 to.array 的地址处（会覆盖原有的值）


### Empty Slice 和 Nil Slice

- Empty:
```
func main() {
    nums := []int{}          // 用类型推到为nums进行了定义和初始化工作。
    renums := make([]int, 0) // make 为renums 进行了定义和初始化工作.

    fmt.Printf("nums: %v, len: %d, cap: %d\n", nums, len(nums), cap(nums))
    fmt.Printf("renums: %v, len: %d, cap: %d\n", renums, len(renums), cap(renums))
}
```

- Nil:
```
func main() {
    var nums []int // 只进行了定义，没有初始化。
}
```
    - Nil Slice 指向的是 nil，Empty Slice 指向的是实际存在的空数组地址。
    - 可以认为Nil Slice 代指不存在的 Slice，Empty Slice 代指空集合。两者所代表的意义是完全不同的。

### Slice 所允许申请的最大容量大小，与当前值类型和当前平台位数有直接关系