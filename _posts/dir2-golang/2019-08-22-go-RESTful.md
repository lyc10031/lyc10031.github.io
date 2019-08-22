---
layout: article
title: RESTful架构及Go语言的实现笔记
tags: 
- GO
- RESTful
- 学习笔记
key: go-REST
aside:
  toc: true
---

[原文地址](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/08.3.md)

本文是我的一些总结和记录。

# RESTful

- RESTful，是目前最为流行的一种互联网软件架构。因为它结构清晰、符合标准、易于理解、扩展方便，所以正得到越来越多网站的采用。
> REST(REpresentational State Transfer)这个概念，首次出现是在 2000年Roy Thomas Fielding（他是HTTP规范的主要编写者之一）的博士论文中，它指的是一组架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是RESTful的。

1. 资源（Resources） REST是"表现层状态转化"，这里表现层指的是"资源"的"表现层"。
> 那么什么是资源呢？就是我们平常上网访问的一张图片、一个文档、一个视频等。这些资源我们通过URI来定位，也就是一个URI表示一个资源。

2. 表现层（Representation）
> 资源是一个真实存在的实体信息，这个信息有多种的表现形式。这个表现形式就是表现层。一段TXT文本信息可以用输出成html,json,xml 等格式，一个图片可以用不同的格式jpg、png等方式展现。

3. 状态转化（state Transfer）
> 一个网站被访问，就会存在客户端和服务端的交互，在交互过程中会有数据和状态的变化，由于HTTP协议是无状态的，所以状态信息必须被保存在服务端，客户端通过HTTP协议里面的4种操作方式GET、POST、PUT、DELETE通知服务端进行状态的更新，数据的增减。

    >    GET用来获取资源.

    >    POST用来新建资源（也可以用于更新资源）.

    >    PUT用来更新资源.

    >    DELETE用来删除资源.

##  RESTful架构的特点：
1. 每一个URI代表一种资源；
2. 客户端和服务器之间，传递这种资源的某种表现层；
3. 客户端通过四个HTTP方法，对服务器端资源进行操作，实现"表现层状态转化"。

Web应用要满足REST最重要的原则是:客户端和服务器之间的交互在每次请求之间是无状态的,客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。此外此请求可以由任何可用服务器回答，这十分适合云计算之类的环境。因为是无状态的，所以客户端可以缓存数据以改进性能。

另一个重要的REST原则是系统分层，这表示组件无法了解除了与它直接交互的层次以外的组件。通过将系统知识限制在单个层，可以限制整个系统的复杂性，从而促进了底层的独立性。

- REST架构图

![Image](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/images/8.3.rest2.png?raw=true){:.rounded}

将REST架构的应用看成一个整体，REST有较强的可扩展性。它还降低了客户端和服务器之间的交互延迟。统一界面简化了整个系统架构，改进了子系统之间交互的可见性。REST简化了客户端和服务器的实现。


## Go 的RESTful 实现

Go没有为REST提供直接支持，可以用net/http 包来自己实现，REST根据不同method(get、post、put、delete)来处理资源，

- HTML标准只能通过链接和表单支持GET和POST。在没有Ajax支持的网页浏览器中不能发出PUT或DELETE命令。

- 有些防火墙会挡住HTTP PUT和DELETE请求，要绕过这个限制，客户端需要把实际的PUT和DELETE请求通过 POST 请求穿透过来。RESTful 服务则要负责在收到的 POST 请求中找到原始的 HTTP 方法并还原。


[第三方库 httprouter 实现了RESTful方式](github.com/julienschmidt/httprouter)
这个库实现了自定义路由和方便的路由规则映射，通过它，我们可以很方便的实现REST的架构。

### 代码

{% highlight golang linenos%}
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/julienschmidt/httprouter"
)

func Index(w http.ResponseWriter, r *http.Request, _ httprouter.Params) {
	fmt.Fprint(w, "Welcome!\n")
}

func Hello(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	fmt.Fprintf(w, "hello, %s!\n", ps.ByName("name"))
}

func getuser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	uid := ps.ByName("uid")
	fmt.Fprintf(w, "you are get user %s", uid)
}

func modifyuser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	uid := ps.ByName("uid")
	fmt.Fprintf(w, "you are modify user %s", uid)
}

func deleteuser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	uid := ps.ByName("uid")
	fmt.Fprintf(w, "you are delete user %s", uid)
}

func adduser(w http.ResponseWriter, r *http.Request, ps httprouter.Params) {
	// uid := r.FormValue("uid")
	uid := ps.ByName("uid")
	fmt.Fprintf(w, "you are add user %s", uid)
}

func main() {
	router := httprouter.New()
    // 不同的method方法，对应不同的函数。
	router.GET("/", Index)
	router.GET("/hello/:name", Hello)

	router.GET("/user/:uid", getuser)
	router.POST("/adduser/:uid", adduser)
	router.DELETE("/deluser/:uid", deleteuser)
	router.PUT("/moduser/:uid", modifyuser)

	log.Fatal(http.ListenAndServe(":8080", router))
}
{% endhighlight %}

## 总结

REST是一种架构风格，汲取了WWW的成功经验：无状态，以资源为中心，充分利用HTTP协议和URI协议，提供统一的接口定义，使得它作为一种设计Web服务的方法而变得流行。在某种意义上，通过强调URI和HTTP等早期Internet标准，REST是对大型应用程序服务器时代之前的Web方式的回归。目前Go对于REST的支持还是很简单的，通过实现自定义的路由规则，我们就可以为不同的method实现不同的handle，这样就实现了REST的架构。

<a href="javascript:scroll(0,0)">-- 返回顶部 --</a>