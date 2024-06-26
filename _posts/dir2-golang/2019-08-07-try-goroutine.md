---
layout: article
title: goroutine学习记录
tags: GO-lang
key: groutine1
aside:
  toc: true
---
# 学习下goroutine的使用方法

## goroutine是Go并行设计的核心。

goroutine说到底其实就是协程，但是它比线程更小，十几个goroutine可能体现在底层就是五六个线程，Go语言内部帮你实现了这些goroutine之间的内存共享。执行goroutine只需极少的栈内存 **(大概是4~5KB)** ，当然会根据相应的数据伸缩。也正因为如此，可同时运行成千上万个并发任务。**goroutine比thread更易用、更高效、更轻便。**

goroutine通过go关键字实现了，其实就是一个普通的函数。
通过关键字go就启动了一个goroutine

{% highlight golang linenos%}
go hello(a, b, c)
{% endhighlight %}

## goroutine 的设计要遵循：不通过共享来通信，而是通过通信来共享。

goroutine 通过channel 来仅需信息的共享。channel可以与Unix shell 中的双向管道做类比：可以通过它发送或者接收值。这些值只能是特定的类型：channel类型。定义一个channel时，也需要定义发送到channel的值的类型。注意，必须*使用make 创建channel*创建并初始化：

{% highlight golang linenos%}
ci := make(chan int)
cs := make(chan string)
cf := make(chan interface{})
{% endhighlight %}

### channel通过操作符<-来接收和发送数据

{% highlight golang linenos%}
ch <- v    // 发送v到channel ch.
v := <-ch  // 从ch中接收数据，并赋值给v
{% endhighlight %}

#### 例1、

{% highlight golang linenos%}
func test1() {
	a := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(a[:len(a)/2], c)
	go sum(a[len(a)/2:], c)

	x, y := <-c, <-c //receive from chan c
	fmt.Println(x, y, x+y)
}

func sum(a []int, c chan int) {
	total := 0
	for _, v := range a {
		total += v
	}
	c <- total
}
{% endhighlight %}
默认情况下，channel接收和发送数据都是阻塞的，除非另一端已经准备好，这样就使得Goroutines同步变的更加的简单，而不需要显式的lock。所谓阻塞，也就是如果读取（value := <-ch）它将会被阻塞，直到有数据接收。其次，任何发送（ch<-5）将会被阻塞，直到数据被读出。无缓冲channel是在多个goroutine之间同步很棒的工具.

### 带缓存的channels Buffered Channels

可以控制channel能存储的元素数量
当写入的数量超过缓存数量时 代码会阻塞，直到其他goroutine从channel 中读取一些元素，腾出空间。
#### 例2、
{% highlight golang linenos%}
func test1() {
	// channel with buffered
	c := make(chan int, 3) //change to 2 will put painc: "fatal error: all goroutines are asleep - deadlock!"
	c <- 1
	c <- 2
	c <- 3
	fmt.Println(<-c)
	fmt.Println(<-c)
	fmt.Println(<-c)
}
{% endhighlight %}
上面的例子中重复读取了多次，比较啰嗦。可以通过range，像操作slice或者map一样操作缓存类型的channel，请看下面的例子

#### 例3、
{% highlight golang linenos%}
func fibonacci() {
	/*
		for i := range c能够不断的读取channel里面的数据，直到该channel被显式的关闭。上面代码我们看到可以显式的关闭channel，生产者通过内置函数close关闭channel。
		关闭channel之后就无法再发送任何数据了，在消费方可以通过语法v, ok := <-ch测试channel是否被关闭。如果ok返回false，那么说明channel已经没有任何数据并且已经被关闭。

		#记住应该在生产者的地方关闭channel，而不是消费的地方去关闭它，这样容易引起panic

		#另外记住一点的就是channel不像文件之类的，不需要经常去关闭，只有当你确实没有任何发送数据了，或者你想显式的结束range循环之类的

	*/
	c := make(chan int, 10)
	go func(n int, c chan int) {
		x, y := 1, 1
		for i := 0; i < n; i++ {
			c <- x // put number to c channel
			x, y = y, x+y
		}
		close(c)
	}(cap(c), c)

	for i := range c { // use range get nember from c channel until c is close()
		fmt.Println(i)
	}
}
{% endhighlight %}

### 使用select操作多个channel

通过select可以监听channel上的数据流动。select默认是阻塞的，只有当监听的channel中有发送或接收可以进行时才会运行，当多个channel都准备好的时候，select是随机的选择一个执行的。
{% highlight golang linenos%}
package main

import (
	"fmt"
)

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)

}

func fibonacci(c, quit chan int) {
	x, y := 1, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit: //从quit channel 中取到了数据。
			fmt.Println("Quit ...")
			return // 退出整个程序
		}
	}
}
{% endhighlight %}

select还有default 语法，select其实就是类似switch的功能，default就是当监听的channel都没有准备好的时候，默认执行的（select不再阻塞等待channel）。

{% highlight golang linenos%}
select {
case i := <-c:
	// use i
default:
	// 当c阻塞的时候执行这里
}
{% endhighlight %}

### channel超时用法

有时候会出现goroutine阻塞的情况，那么我们如何避免整个程序进入阻塞的情况呢？我们可以利用select来设置超时，通过如下的方式实现：

{% highlight golang linenos%}
func main() {
	c := make(chan int)
	o := make(chan bool)
	go func() {
		for {
			select {
				case v := <- c:
					println(v)
				case <- time.After(5 * time.Second): // time.After() 返回一个channel case获取到channel信息后即表示达到超时设置的时间。
					println("timeout")
					o <- true // 向o channel 写数据，在主函数中获取o channel 的信息。
					break
			}
		}
	}()
	<- o
}
{% endhighlight %}

### WaitGroup和Mutex

{% highlight golang linenos%}
package main

import (
	"fmt"
	"sync"
)

var wg = sync.WaitGroup{}
var m = sync.Mutex{}
var conunt = 0

func main() {
	for i := 0; i <= 10; i++ {
		wg.Add(2)
		m.Lock() // 加锁
		go sayhi()
		m.Lock() // 加锁
		go increment()
	}
	wg.Wait() // 等待goroutine运行完毕
}

func sayhi() {
	fmt.Printf("Hello #%v\n", conunt)
	m.Unlock() // 解锁
	wg.Done()
}

func increment() {
	conunt++
	m.Unlock() // 解锁
	wg.Done()
}

{% endhighlight %}
Mutex 加锁后 将会按照顺序输出信息，不加锁会乱序输出。
{% highlight golang linenos%}
func main() {
    wg := sync.WaitGroup{}
    wg.Add(100)
    for i := 0; i < 100; i++ {
        go f(i, &wg)
    }
    wg.Wait()
}

// 一定要通过指针传值，不然进程会进入死锁状态
func f(i int, wg *sync.WaitGroup) { 
    fmt.Println(i)
    wg.Done()
}
{% endhighlight %}

## 关于WaitGroup.Wait()与timeout
See ***Timeout for WaitGroup.Wait()*** [stackoverflow](https://stackoverflow.com/questions/32840687/timeout-for-waitgroup-wait)

Mostly your solution you posted [below](https://stackoverflow.com/a/32840688/1705598) is as good as it can get. Couple of tips to improve it:

+ Alternatively you may close the channel to signal completion instead of sending a value on it, a receive operation on a closed channel can always proceed immediately.

+ And it's better to use defer statement to signal completion, it is executed even if a function terminates abruptly.

+ Also if there is only one "job" to wait for, you can completely omit the WaitGroup and just send a value or close the channel when job is complete (the same channel you use in your select statement).

+ Specifying 1 second duration is as simple as: timeout := time.Second. Specifying 2 seconds for example is: timeout := 2 * time.Second. You don't need the conversion, time.Second is already of type time.Duration, multiplying it with an untyped constant like 2 will also yield a value of type time.Duration.

I would also create a helper / utility function wrapping this functionality. Note that WaitGroup must be passed as a pointer else the copy will not get "notified" of the WaitGroup.Done() calls. Something like:

// waitTimeout waits for the waitgroup for the specified max timeout.
// Returns true if waiting timed out.
{% highlight golang linenos%}
func waitTimeout(wg *sync.WaitGroup, timeout time.Duration) bool {
    c := make(chan struct{})
    go func() {
        defer close(c)
        wg.Wait()
    }()
    select {
    case <-c:
        return false // completed normally
    case <-time.After(timeout):
        return true // timed out
    }
}
{% endhighlight %}
Using it:
{% highlight golang linenos%}
if waitTimeout(&wg, time.Second) {
    fmt.Println("Timed out waiting for wait group")
} else {
    fmt.Println("Wait group finished")
}
{% endhighlight %}
## runtime goroutine

### runtime包中有几个处理goroutine的函数：
+ Goexit 

    退出当前执行的goroutine，但是defer函数还会继续调用

+ Gosched 

    让出当前goroutine的执行权限，调度器安排其他等待的任务运行，并在下次某个时候从该位置恢复执行。

+ NumCPU

    返回 CPU 核数量

+ NumGoroutine 

    返回正在执行和排队的任务总数

+ GOMAXPROCS 

    用来设置可以并行计算的CPU核数的最大值，并返回之前的值。 


<a href="javascript:scroll(0,0)">-- 返回顶部 --</a>


<!-- <script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>  -->


 <!-- <span id="busuanzi_container_site_pv">
    本站总访问量<span id="busuanzi_value_site_pv"></span>次
</span>

 <span id="busuanzi_container_site_uv">
  本站访客数<span id="busuanzi_value_site_uv"></span>人次
</span>

 <span id="busuanzi_container_page_pv">
  本文总阅读量<span id="busuanzi_value_page_pv"></span>次
</span> -->


<script src="https://jerry-cdn.b0.upaiyun.com/hit-kounter/hit-kounter-0.1.1.js"></script>

浏览量：<span data-hk-page="current"> - </span>次<span class="pause"> | </span>