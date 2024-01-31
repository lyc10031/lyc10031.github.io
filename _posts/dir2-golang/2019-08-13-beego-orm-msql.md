---
layout: article
title: '使用beego-orm 操作mysql数据库'
key: 12
# category: Document
tags:
- golang
- beego-orm
- mysql
aside:
  toc: true
---

[原文章地址在这里](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/05.5.md)


## 代码:
{% highlight golang linenos%}
package main

// 接下来我们的例子采用前面(test1/main.go)的数据库表User，现在我们建立相应的struct
import (
	"time"

	"github.com/astaxie/beego/orm"     // 采用了Go style方式对数据库进行操作，实现了struct到数据表记录的映射.
	_ "github.com/go-sql-driver/mysql" // _ 代表只执行包里面的init部分 完成初始化工作.
)

type Userinfo struct {
	Uid        int `orm:"PK"` //如果表的主键不是id，那么需要加上pk注释，显式的说这个字段是主键
	Username   string
	Departname string
	Created    time.Time
}

type User struct {
	Uid     int `orm:"PK"` // 如果表的主键不是id，那么需要加上pk注释，显式的说这个字段是主键
	Name    string
	Profile *Profile `orm:"rel(one)"`      // OneToOne relation
	Post    []*Post  `orm:"reverse(many)"` //  设置一对多的反向关系
}

type Profile struct {
	Id   int
	Age  int16
	User *User `orm:"reverse(one)"` // 设置一对一反向关系(可选)
}

type Post struct {
	Id    int
	Title string
	User  *User  `orm:"rel(fk)"`
	Tags  []*Tag `orm:"rel(m2m)"`
}

type Tag struct {
	Id    int
	Name  string
	Posts []*Post `orm:"reverse(many)"`
}

func init() {
	// 设置默认数据库
	orm.RegisterDataBase("default", "mysql", "root:123456@tcp(localhost:3306)/test?charset=utf8", 30)
	// 需要在init中注册定义的model
	orm.RegisterModel(new(Userinfo), new(User), new(Profile), new(Post), new(Tag))
	// 创建table
	orm.RunSyncdb("default", false, true)

	orm.Debug = true // 打印调试信息
}

func main() {

}
{% endhighlight %}

以下都是在main函数中进行mysql操作的方法:

## 插入数据
{% highlight golang linenos%}
	o := orm.NewOrm()
	var user Userinfo
	user.Username = "xxx2"
	user.Departname = "software-xxx2"
	id, err := o.Insert(&user)
	/*
	 	[INSERT INTO `userinfo` (`uid`, `username`, `departname`, `created`) VALUES (?, ?, ?, ?)] - `0`, `leiyongcheng`, `software-engineer`, `<nil>`
	*/
	if err != nil {
	 	log.Fatalf("ERROR: %v\n", err)
	} else {
	 	fmt.Println(id)
	}
{% endhighlight %}

## 同时插入多个对象:InsertMulti
{% highlight golang linenos%}
	/*
			类似sql语句

		insert into table (name, age) values("slene", 28),("astaxie", 30),("unknown", 20)
	*/
	o := orm.NewOrm()
	users := []User{
	 	{Name: "slene"},
	 	{Name: "astaxie"},
	 	{Name: "unknown"},
	 	...
	}
	successNums, err := o.InsertMulti(100, users)
{% endhighlight %}

## 更新数据
{% highlight golang linenos%}
	o := orm.NewOrm()
	user := Userinfo{Uid: 1} // 类似sql 语言中的where 这里Uid 是主键
	if o.Read(&user) == nil {
	 	/*
	 		sql：
	 		[SELECT `uid`, `username`, `departname`, `created` FROM `userinfo` WHERE `uid` = ? ] - `1`
	 		这里进行的操作是
	 		先从表中以uid=1为主键查找userinfo的数据

	 	*/
	 	user.Username = "new-name" // 将uid = 1 的数据的Username 更新成new-name
	 	if num, err := o.Update(&user); err == nil {
	 		/*
	 			sql：
	 			[UPDATE `userinfo` SET `uid` = ?, `username` = ?, `departname` = ?, `created` = ? WHERE `uid` = ?] -
	 			`1`, `new-name`, `software-engineer`, `<nil>`, `1`
	 		*/
	 		fmt.Println(num)
	 	}
	}
{% endhighlight %}

## 查询数据 
### 例子1：
{% highlight golang linenos%}
	o := orm.NewOrm()
	user := Userinfo{Uid: 2}
	/*
		可以通过修改Uid 查询一条不存在的信息，能够显示查询不到
	 	查询存在的Uid 编号。能够查到数据库中存放的信息。
	*/
	err := o.Read(&user)
	/*
	 	sql:
	 	[SELECT `uid`, `username`, `departname`, `created` FROM `userinfo` WHERE `uid` = ? ] - `2`
	*/
	if err == orm.ErrNoRows {
	    fmt.Println("查询不到")
	} else if err == orm.ErrMissPK {
	 	fmt.Println("找不到主键")
	} else {
	 	fmt.Println(user.Uid, user.Username)
	}
{% endhighlight %}

### 例子2：
{% highlight golang linenos%}
	o := orm.NewOrm()
	var user Userinfo

	qs := o.QueryTable(user)      // 返回 QuerySeter
	qs.Filter("id", 3)            // WHERE id = 1
	qs.Filter("username", "xxx2") // WHERE username=xxx2

	qs.Filter("profile__age__in", 18, 20)
	// // WHERE profile.age IN (18, 20)

	qs.Filter("profile__age__in", 18, 20).Exclude("profile__lt", 1000)
	// // WHERE profile.age IN (18, 20) AND NOT profile_id < 1000
{% endhighlight %}

## 获取多条数据

### 例子1，根据条件age>17，获取20位置开始的10条数据的数据
{% highlight golang linenos%}
	var allusers []User
	qs.Filter("profile__age__gt", 17)
	// // WHERE profile.age > 17
{% endhighlight %}
### 获取多条数据  例子2，limit默认从10开始，获取10条数据
{% highlight golang linenos%}
    //例子2，limit默认从10开始，获取10条数据
	qs.Limit(10, 20)
	// // LIMIT 10 OFFSET 20 注意跟SQL反过来的
{% endhighlight %}

## 删除数据
删除单条数据
{% highlight golang linenos%}
	if num, err := o.Delete(&Userinfo{Uid: 1}); err == nil {
	 	fmt.Println(num)
	}
	//Delete 操作会对反向关系进行操作，此例中 Post 拥有一个到 User 的外键。删除 User 的时候。
	// 如果 on_delete 设置为默认的级联操作，将删除对应的 Post
{% endhighlight %}
##  关联查询
{% highlight golang linenos%}
	type Post struct {
	 	Id    int    `orm:"auto"`
	 	Title string `orm:"size(100)"`
	 	User  *User  `orm:"rel(fk)"` // 将另一个表作引入当前struct ，查询的时候 All（&posts） 会应用到所有posts 表中。
	}

	var posts []*Post
	qs := o.QueryTable("post")
	num, err := qs.Filter("User__Name", "slene").All(&posts)
{% endhighlight %}
## GroupBy和Having
针对有些应用需要用到group by的功能，beego orm也提供了一个简陋的实现
{% highlight golang linenos%}
	qs.OrderBy("id", "-profile__age")
	// ORDER BY id ASC, profile.age DESC

	qs.OrderBy("-profile__age", "profile")
	// ORDER BY profile.age DESC, profile_id ASC

	// 上面的代码中出现了两个新接口函数

	// GroupBy:用来指定进行groupby的字段

	// Having:用来指定having执行的时候的条件
{% endhighlight %}

## 最后，使用原始sql命令：
### 简单示例:
    {% highlight golang linenos%}
	o := orm.NewOrm()
	var r orm.RawSeter
	r = o.Raw("UPDATE user SET name = ? WHERE name = ?", "testing", "xxx2")

{% endhighlight %}

### 复杂的原生sql使用:
{% highlight golang linenos%}

func (m *User) Query(name string) user []User {
 	var o orm.Ormer
 	var rs orm.RawSeter
 	o = orm.NewOrm()
 	rs = o.Raw("SELECT * FROM user" +
	"WHERE name=? AND uid>10 "+
 	"ORDER BY uid DESC "+
 	"LIMIT 100", name)
 	//var user []User
 	num, err := rs.QueryRows(&user)
 	if err != nil {
 		fmt.Println(err)
 	} else {
 		fmt.Println(num)
 		//return user
 	}
 	return
}

{% endhighlight %}

<a href="javascript:scroll(0,0)">-- 返回顶部 --</a>