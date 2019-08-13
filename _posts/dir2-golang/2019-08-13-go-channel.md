---
layout: article
title: Learn Go channel
tags: 
- GO-lang
- channel
key: 12
category: Document
aside:
  toc: true
---

## 学习Go channels 的记录
最近在YouTube上看一个GO 的学习视频。记录下学习过程的知识点。
链接在这里：

[Learn Go Programming - Golang Tutorial for Beginners](https://www.youtube.com/watch?v=YS4e4q9oBaU) 

因为太长了，YouTube不能自动生成字幕。啃的生肉，有理解不恰当地方的欢迎指正

废话不多说直接上代码:
{% highlight golang linenos%}
package main

import (
	"fmt"
	"time"
)

const (
	logInfo   = "INFO"
	logWaring = "WARNING"
	logError  = "ERROR"
)

type logEntry struct {
	time      time.Time
	serverity string
	message   string
}

var logCh = make(chan logEntry, 50)
var doneCh = make(chan struct{}) 
// 信令通道。使用struct{} 作为信令通道是消耗最小的。不会请求额外的内存资源

func main() {
	go logger()
	logCh <- logEntry{time.Now(), logInfo, "App is starting"}

	logCh <- logEntry{time.Now(), logInfo, "App is shuting down"}
	time.Sleep(100 * time.Millisecond)
	doneCh <- struct{}{} // use this is no memory cost
}

func logger() {
	for {
		select { // 阻塞 ，直到channel 中有信号传入。开始进行匹配
		case entry := <-logCh:
			fmt.Printf("%v - [%v]%v\n", entry.time.Format("2006-01-02 15:04:05"), entry.serverity, entry.message)
		case <-doneCh:
        // 从doneCh 中取到信号，说明main goroutine 发出了结束信息。 break挑出循环，结束程序
			break
        default: 
        // 如果不加default 部分，程序会一直阻塞直到有（信号、信息）传入channel 中。
        // 加了default 后，在channel 中无数据时，将会执行default部分。
            //something 
		}
	}
}

{% endhighlight %}


两种从channel中获取数据的方法:
{% highlight golang linenos%}

package main

import (
	"fmt"
	"sync"
)

var wg sync.WaitGroup

func main() {
	ch := make(chan int)
	wg.Add(2)
	go func(ch <-chan int) { // write only channel

        // 第一种 使用range 获取 类似slice 或 map 行为
		// for i := range ch {
		// 	fmt.Println(i)
		// }

        // 第二种 适合多channel 获取不同channel 的数据 , 与上面例子中使用select 获取channel数据类似
		for {
			if i, ok := <-ch; ok {
				fmt.Println(i)
			} else {
				break
			}
		}
		wg.Done()
	}(ch)
	go func(ch chan<- int) { // read only channel
		ch <- 42
		ch <- 901
		close(ch)
		wg.Done()
	}(ch)
	wg.Wait()
}

{% endhighlight %}