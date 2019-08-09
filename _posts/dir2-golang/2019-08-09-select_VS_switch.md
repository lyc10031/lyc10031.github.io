---
layout: article
title: Go switch VS select
tags: GO-lang
date: 2019-08-07 07:58:05 +0800
aside:
  toc: true
---


## switch is used to make a decision based on a variable value of any type.

switch用于根据任何类型的变量值做出决定。

+ Read this for more details:

Go's switch is more general than C's. The expressions need not be constants or even integers, the cases are evaluated top to bottom until a match is found, and if the switch has no expression it switches on true. It's therefore possible—and idiomatic—to write an if-else-if-else chain as a switch.

Here is one example: [Go Playground](https://play.golang.org/p/g7F7d7OG43)

Go的 switch 比C的更普遍。 表达式不必是常量甚至是整数，在找到匹配项之前从上到下评估案例，
如果switch没有表达式，则将其置为true。 因此，将if-else-if-else判断链使用switch编写是可能的，也是惯用的。
### switch例子
{% highlight golang linenos%}
package main

import (
    "fmt"
    "runtime"
)

func main() {
    fmt.Print("Go runs on ")
    switch os := runtime.GOOS; os {
    case "darwin":
        fmt.Println("OS X.")
    case "linux":
        fmt.Println("Linux.")
    default:
        // freebsd, openbsd,
        // plan9, windows...
        fmt.Printf("%s.", os)
    }
}
{% endhighlight %}


### 再看一个例子：

{% highlight golang linenos%}
package main

import (
	"fmt"
	"strconv"
)

type Element interface{}

type List []Element

type Person struct {
	name string
	age  int
}

// 定义了String方法，实现了fmt.Stringer
func (p Person) String() string {
	return "(name: " + p.name + " - age: " + strconv.Itoa(p.age) + " years)"
}

func main() {
	list := make(List, 3)
	list[0] = 0
	list[1] = "Hello"
	list[2] = Person{"Dennis", 20}

	// for index, element := range list {
	// 	if value, ok := element.(int); ok {  //使用if-else-if-else 方式
	// 		fmt.Printf("list[%d] is an int and it's value is %d\n", index, value)
	// 	} else if value, ok := element.(string); ok {
	// 		fmt.Printf("list[%d] is a string and it's value is %s\n", index, value)
	// 	} else if value, ok := element.(Person); ok {
	// 		fmt.Printf("list[%d] is a Person and it's value is %s\n", index, value)
	// 	} else {
	// 		fmt.Printf("list[%d] is of different type \n", index)
	// 	}
	// }
	for index, element := range list {
		switch value := element.(type) { //使用switch方式
		case int:
			fmt.Printf("list[%d] is an int and it's value is %d\n", index, value)
		case string:
			fmt.Printf("list[%d] is a string and it's value is %s\n", index, value)
		case Person:
			fmt.Printf("list[%d] is a Person and it's value is %d\n", index, value)
		default:
			fmt.Printf("list[%d] is of a different type\n", index)
		}
	}
}
{% endhighlight %}

## The select statement lets a goroutine wait on multiple communication operations.

select语句允许goroutine等待多个通信操作。

A select blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready. Here is one example: [Go Playground](https://play.golang.org/p/ipOXPcmWcL)


一个select块会查找所有case直到其中一个case可以运行，然后执行该case。 如果多个case准备就绪，它会随机选择一个。 
所有case都不可运行 就执行default内容。

### select 例子
{% highlight golang linenos%}
package main

import (
    "fmt"
    "time"
)

func main() {
    tick := time.Tick(100 * time.Millisecond)
    boom := time.After(500 * time.Millisecond)
    for {
        select {
        case <-tick:
            fmt.Println("tick.")
        case <-boom:
            fmt.Println("BOOM!")
            return
        default:
            fmt.Println("    .")
            time.Sleep(50 * time.Millisecond)
        }
    }
}
{% endhighlight %}