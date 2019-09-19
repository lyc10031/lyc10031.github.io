---
layout: article
title: Go 脚本记录说明
tags: 
- GO
- 学习记录
key: go-scripts
aside:
  toc: true
---

## 翻转一个整数
- 输入123,返回321；输入-321返回-123
- 输入的整数要求是一个 32bit 有符号数，如果反转后溢出，则输出 0

{% highlight golang linenos%}
func reverse(x int) (num int) {
	for x != 0 {
		num = num*10 + x%10
		x = x / 10
	}
	// 使用 math 包中定义好的最大最小值
	if num > math.MaxInt32 || num < math.MinInt32 {
		return 0
	}
	return
}
{% endhighlight %}
- 
    - 2、3、4行进行翻转操作。
    - 输入123，第二行先判断是否为0；不为0 就进入循环内部。
    - 第一次循环：num等于 0 * 10 + x%10 = 3
    - x/10 取整数部分，此时为12。
    - 第二次循环： x 不为0 进入循环。
    - num 等于3 * 10 + 12 % 10 = 32
    - x 等于 12 / 10 此时为1。
    - 第三次循环：x不为0 进入循环。
    - num 等于32 * 10 + 1 % 10 = 321
    - x 等于 1 / 10 此时为0。
    - for判断后将跳出循环。返回num 321。


## 两数之和
- 给定一个整数数组和一个目标值，找出数组中和为目标值的两个数。

你可以假设每个输入只对应一种答案，且同样的元素不能被重复利用。
  - 例：
      > 给定 nums = [2, 7, 11, 15], target = 9

      >  因为 nums[0] + nums[1] = 2 + 7 = 9
      
      >  所以返回 [0, 1]


{% highlight golang linenos%}
func add2(a []int, b int) []int {
	m := make(map[int]int)
	for index, value := range a {
		tmp := b - value
		if _, ok := m[tmp]; ok {
			// return []int{m[tmp], index} // 找出目标值的索引
			return []int{tmp, value} // 找出目标值
		} else {
			m[value] = index
		}
	}
	return nil
}
{% endhighlight %}

- 
    - 时间复杂度：O(n),空间复杂度：O(n)
    - 建立哈希表(map) 遍历slice，以值为key，索引为value。
    - 判断target - slice\[n\] 的结果是否存在于map 中，如果存在即输出结果。
    - 不存在就继续遍历。

<a href="javascript:scroll(0,0)">-- 返回顶部 --</a>