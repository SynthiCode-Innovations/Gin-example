# Golang Gin Web Framework

Created: April 13, 2023 10:25 PM
Reviewed: No

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled.png)

## 1. 第一个官方的demo和用法

---

1. 导入gin框架包
2. 调用gin.Default()生成gin引擎对象
3. r.GET方法参数 / urlpath参数 第二个参数是路由处理函数的参数是封装的gin.Context上下文对象
4. 函数里面c.JSON返回对应的状态码以及response

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{
			"msg": "pong",
		})
	})

	r.Run()
}
```

## 2.Get 请求方法如何取值并返回打印到浏览器

---

```go
package main

import "github.com/gin-gonic/gin"

func Handle(c *gin.Context) {
	// 利用gin.Context下的 c.Query() 和c.DefaultQuery()取值url后面的Get参数
	// example: https://baidu.com/?username=&&age=20
	name := c.Query("name")
	age := c.Query("age")
	// 并且返回对应response
	c.String(200, "name: %s, age:%s", name, age)
}

func main() {
	r := gin.Default()

	r.GET("/", Handle)

	r.Run()
}
```

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%201.png)

## 3.Post方法如何进行取值

---

```go
package main

import "github.com/gin-gonic/gin"

func HandldePost(c *gin.Context) {
	//post方法进行取值 post提交方法在payload体里面进行提交
	//后面也可以加上url的参数 http://localhost:8080/?name=liubaobao&&age=20&&kind=doctor
	// kind doctor
	// name = liubaobao
	// age = 20

	name := c.Query("name")
	age := c.Query("age")
	kind := c.DefaultQuery("kind", "unemployed")
	c.String(200, "name=%s, age=%s, kind=%s", name, age, kind)
}

func main() {
	r := gin.Default()
	r.POST("/", HandldePost)
	r.Run()
}
```

### 结果:

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%202.png)

## 4.Post方法表单form如何进行取值

```go
package main

import "github.com/gin-gonic/gin"

func HandlePostForm(c *gin.Context) {
	// form表单提交进行取值key为name
	name := c.PostForm("name")
	// form表单提交进行取值,如果没有给一个默认值20
	age := c.DefaultPostForm("age", "20")
	// form表单提交进行取值key为kind
	kind := c.PostForm("kind")
	c.String(200, "name=%s, age=%s, kind=%s", name, age, kind)

}

func main() {
	r := gin.Default()
	r.POST("/postform", HandlePostForm)
	r.Run()
}
```

### 结果:

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%203.png)

# 5. 通过html页面进行Get然后进行提交如何进行加载HTML模版并进行取值

---

html页面代码

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width" />
    <title>Welcome</title>
  </head>
  <body>
    <form action="/submit" method="post" accept-charset="utf-8">
      <input type="text" value="liubb" name="name" />
      <input type="text" value="20" name="age" />
      <input type="text" value="doctor" name="kind"/>
      <button type="submit">提交</button>
    </form>
  </body>
</html>
```

golang 代码

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func HandlePostHtml(c *gin.Context) {
	name := c.PostForm("name")
	age := c.PostForm("age")
	kind := c.PostForm("kind")
	c.String(http.StatusOK, "name=%s, age=%s, kind=%s", name, age, kind)

}

func HandleGetHtml(c *gin.Context) {
	c.HTML(http.StatusOK, "welcome.html", "")
}

func main() {
	r := gin.Default()
	r.GET("/", HandleGetHtml)
	r.POST("/submit", HandlePostHtml)
	r.LoadHTMLFiles("welcome.html")
	r.Run()
}
```

## 结果:

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%204.png)

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%205.png)

# 6.数据表单多个值进行绑定注册的例子如下

html代码

```go
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width" />
    <title>Register</title>
  </head>
  <body>
    <form action="/toregister" method="post" accept-charset="utf-8">
      <input type="text" value="" name="name" id="name"/> 姓名
      <input type="text" value="" name="password" id=""/> 密码
      <input type="checkbox" name="hondy" value="Java" id=""> Java
      <input type="checkbox" name="hondy" value="Golang" id=""> Golang
      <input type="checkbox" name="hondy" value="Python" id=""> Python
      <button type="submit">提交</button>
    </form>
  </body>
</html>
```

golang-main代码

```go
package main

import (
	"gin_example/models"
	"net/http"

	"github.com/gin-gonic/gin"
)

func HandleRegister(c *gin.Context) {
	c.HTML(http.StatusOK, "register.html", nil)
}

func HandleToRegister(c *gin.Context) {
	var user models.User
	c.ShouldBind(&user)
	c.String(http.StatusOK, "name: %s, password:%s, hony:%v", user.Name, user.Password, user.Hondy)
}

func main() {
	r := gin.Default()
	r.GET("/register", HandleRegister)
	r.POST("/toregister", HandleToRegister)
	r.LoadHTMLFiles("register.html")
	r.Run()
}
```

golang-models代码

```go
package models

// 用结构体的tag进行反射查找提交表单的name值
type User struct {
	Name     string   `form:"name"`
	Password string   `form:"password"`
	Hondy    []string `form:"hondy"`
}
```

## 结果:

请求url /register

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%206.png)

提交请求/toregister

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%207.png)

# 6. 引入bootstrap框架的css和js如何进行访问并取值?

---

1. 在当前项目下创建模版文件目录和静态文件目录

```go
$: mkdir templates/
$: mkdir static/
```

1. 下载bootstrap前端静态文件: [https://v5.bootcss.com/docs/getting-started/download/](https://v5.bootcss.com/docs/getting-started/download/)
2. 下载的zip包解压到static目录下 如下:

```go
static (main*) » tree                                                                                                                                                                                                                                                           ~/code/gin_example/static
.
├── css
│   ├── bootstrap-grid.css
│   ├── bootstrap-grid.css.map
│   ├── bootstrap-grid.min.css
│   ├── bootstrap-grid.min.css.map
│   ├── bootstrap-grid.rtl.css
│   ├── bootstrap-grid.rtl.css.map
│   ├── bootstrap-grid.rtl.min.css
│   ├── bootstrap-grid.rtl.min.css.map
│   ├── bootstrap-reboot.css
│   ├── bootstrap-reboot.css.map
│   ├── bootstrap-reboot.min.css
│   ├── bootstrap-reboot.min.css.map
│   ├── bootstrap-reboot.rtl.css
│   ├── bootstrap-reboot.rtl.css.map
│   ├── bootstrap-reboot.rtl.min.css
│   ├── bootstrap-reboot.rtl.min.css.map
│   ├── bootstrap-utilities.css
│   ├── bootstrap-utilities.css.map
│   ├── bootstrap-utilities.min.css
│   ├── bootstrap-utilities.min.css.map
│   ├── bootstrap-utilities.rtl.css
│   ├── bootstrap-utilities.rtl.css.map
│   ├── bootstrap-utilities.rtl.min.css
│   ├── bootstrap-utilities.rtl.min.css.map
│   ├── bootstrap.css
│   ├── bootstrap.css.map
│   ├── bootstrap.min.css
│   ├── bootstrap.min.css.map
│   ├── bootstrap.rtl.css
│   ├── bootstrap.rtl.css.map
│   ├── bootstrap.rtl.min.css
│   └── bootstrap.rtl.min.css.map
└── js
    ├── bootstrap.bundle.js
    ├── bootstrap.bundle.js.map
    ├── bootstrap.bundle.min.js
    ├── bootstrap.bundle.min.js.map
    ├── bootstrap.esm.js
    ├── bootstrap.esm.js.map
    ├── bootstrap.esm.min.js
    ├── bootstrap.esm.min.js.map
    ├── bootstrap.js
    ├── bootstrap.js.map
    ├── bootstrap.min.js
    └── bootstrap.min.js.map

2 directories, 44 files
static (main*) » ls                                                                                                                                                                                                                                                             ~/code/gin_example/static
css js
static (main*) »
```

1. templates 目录

```go
templates (main*) » ll -sh                                                                                                                                                                                                                                              ~/code/gin_example/templates 127 ↵
total 8
8 -rw-r--r--  1 baobao  staff   1.9K  4 15 16:48 index.html
```

代码如下:

```go
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width" />
    <title>Index</title>
    <link href="/static/css/bootstrap.min.css" rel="stylesheet">
    <script src="/static/js/bootstrap.bundle.min.js"></script>
  </head>
  <body>
    <div class="container">
    <form class="row g-3" action="/post" method="post">
      <div class="col-md-6">
        <label for="inputEmail4" class="form-label">Username</label>
        <input type="username" name ="username" class="form-control" id="inputEmail4">
      </div>
      <div class="col-md-6">
        <label for="inputPassword4" class="form-label">Password</label>
        <input type="password" name="password" class="form-control" id="inputPassword4">
      </div>
      <div class="col-12">
        <label for="inputAddress" class="form-label">Address</label>
        <input type="text" name="address" class="form-control" id="inputAddress" placeholder="1234 Main St">
      </div>
      <div class="col-md-6">
        <label for="inputCity" class="form-label">City</label>
        <input type="text" name="city" class="form-control" id="inputCity">
      </div>
      <div class="col-md-4">
        <label for="inputState" class="form-label">State</label>
        <select name="state" id="inputState" class="form-select">
          <option value="0" selected>Choose1</option>
          <option value="1" >Choose2</option>
        </select>
      </div>
      <div class="col-12">
        <div class="form-check">
          <input class="form-check-input" type="checkbox" id="gridCheck" name="gridcheck">
          <label class="form-check-label" for="gridCheck">
            Check me out
          </label>
        </div>
      </div>
      <div class="col-12">
        <button type="submit" class="btn btn-primary">Sign in</button>
      </div>
    </form>
   <div class="card-body">
  </div>
    </div>
  </body>
</html>
```

1. main.go代码如下:

```go
package main

import (
	"gin_example/models"
	"net/http"

	"github.com/gin-gonic/gin"
)

func GetBootstrapHtml(c *gin.Context) {
	// 返回静态页面index.html
	c.HTML(http.StatusOK, "index.html", nil)
}

func PostBootstrapHtml(c *gin.Context) {
	// 定义一个新的usermodel 结构体
	var usermodel models.User
	// 进行绑定赋值给usermodel
	c.ShouldBind(&usermodel)
	c.String(http.StatusOK, "usermodel=%v", usermodel)
}

func main() {
	r := gin.Default()
	r.GET("/get", GetBootstrapHtml)
	r.POST("/post", PostBootstrapHtml)
	// 加载静态文件目录, 可供浏览器端进行访问
	r.Static("/static", "static")
	// 加载静态模版html页面
	r.LoadHTMLGlob("templates/*")
	r.Run()

}
```

### 结果如下:

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%208.png)

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%209.png)

# [7.](http://7.ni)如何使用中间件?

```go
package main

import (
	"fmt"

	"github.com/gin-gonic/gin"
)

func HandlerFuncA(c *gin.Context) {
	fmt.Println("HandlerFuncA....")
}

func RetHandlerFuncB() gin.HandlerFunc {
	return func(c *gin.Context) {
		fmt.Println("HandlerFuncB...")
	}
}

func HandlerFuncC(c *gin.Context) {
	fmt.Println("HandlerFuncC....")
	c.String(200, "Mrs %s WelCome to GinframeWork!", "liubaobao")
}

func main() {
	r := gin.Default()
	/*
		engine 返回的对象使用Use方法进行中间件加载,传入的一个不固定参数是gin.HandlerFunc
		gin.HandlerFunc的源码类型是 type HandlerFunc func(*Context) Context是gin框架里面封装的 http request和response的结构体
		func (engine *Engine) Use(middleware ...HandlerFunc)
	*/
	// 方式一
	r.Use(HandlerFuncA)
	// 方式二
	// 这里执行了一个函数返回了一个func(*gin.Context),效果和方式一是一样的
	r.Use(RetHandlerFuncB())
	r.GET("/", HandlerFuncC)
	r.Run()
}
```

结果如下

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%2010.png)

# 8. 在gin框架里面如何使用BasicAuth?

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

// 定义用户信息的map gin.H类型底层源码的类型就是 map[string]any
var users gin.H = gin.H{
	"nick": gin.H{
		"id":   1,
		"name": "nick",
		"age":  20,
	},
	"tom": gin.H{
		"id":   2,
		"name": "tom",
		"age":  21,
	},
	"jerry": gin.H{
		"id":   3,
		"name": "jerry",
		"age":  22,
	},
}

// 这个路由处理函数是为了测试不访问/v1/admin这个uri是否会验证
func HandlerA(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{
		"msg": "successfully!",
	})
}

// 这个路由处理函数是为了测试访问/v1/admin这个uri是否会验证,并且验证通过是否返回用户的信息详情
func HandlerB(c *gin.Context) {
	username := c.MustGet(gin.AuthUserKey).(string)
	if _, ok := users[username]; ok {
		c.JSON(http.StatusOK, gin.H{
			"userprofile": users[username],
		})
	}
}

func main() {
	r := gin.Default()
	r.GET("/welcome", HandlerA)
	// 路由组里面访问 /v1/admin 都会经过gin.BasicAuth 这个函数进行处理 对输入的用户名和密码和gin.Accounts进行校验
	g := r.Group("/v1/admin", gin.BasicAuth(
		gin.Accounts{
			"nick":  "nick123",
			"tom":   "tom123",
			"jerry": "jerry123",
		},
	))
	g.GET("/", HandlerB)
	r.Run()
}
```

结果如下
1. 浏览器截图

访问url localhost:8080/welcome截图如下, 并没有经过验证

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%2011.png)

访问url [localhost:8080/v1/admin](http://localhost:8080/v1/admin) 截图如下, 并经过验证, 输入错了会一直输入:

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%2012.png)

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%2013.png)

# 9. 如何使用Cookie和Session?

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

func HandlerGetSetCookie(c *gin.Context) {
	// 获取cookie的对应key的值
	id, err := c.Cookie("id")
	// 假如没有获取到的话就设置cookie
	if err != nil {
		c.SetCookie("id", "abcdefghi", 60*60, "/", "localhost:8080", false, false)
	}
	c.JSON(http.StatusOK, gin.H{
		"id": id,
	})
}

func main() {
	r := gin.Default()
	r.GET("/", HandlerGetSetCookie)
	r.Run()
}
```

结果如下:

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%2014.png)

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%2015.png)

**Session**

```go
package main

import (
	"net/http"

	"github.com/gin-contrib/sessions"
	"github.com/gin-contrib/sessions/cookie"
	"github.com/gin-gonic/gin"
)

func main() {

	r := gin.Default()

	// new 存储对象
	store := cookie.NewStore([]byte("secret"))
	// 使用中间件进行每个路由函数进行session 会话的session对象存入到上下文的header里面
	r.Use(sessions.Sessions("MySession", store))
	// 路由进入到路由处理函数然后可以进行校验, 可以拓展一下通过路由分组进行处理会话状态或者登录状态
	r.GET("/welcome", func(c *gin.Context) {
		session := sessions.Default(c)
		// 然后进入摸个路由函数进行session的一个校验操作
		if session.Get("hello") != "world!" {
			session.Set("hello", "world")
			session.Save()
		}

		c.JSON(http.StatusOK, gin.H{
			"hello": session.Get("hello"),
		})

	})
	r.Run()

}
```

:结果截图:

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%2016.png)

# 10.GinFrameWork如何进行数据库操作的初始化

```go
package main

import (
	"fmt"

	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

var Db *gorm.DB

func initDb() {
	// 定义连接数据库的主要信息: host, port, username, password, database
	host := "127.0.0.1"
	port := 3306
	username := "root"
	password := "123456"
	database := "test"
	// dsn 连接串信息
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=utf8&&parseTime=True&loc=Local", username, password, host, port, database)
	var err error
	// gorm.Open() 返回gorm.Db指针类型信息
	Db, err = gorm.Open(mysql.Open(dsn), &gorm.Config{
		// 是否开启日志记录 logger.Info info信息进行记录
		Logger: logger.Default.LogMode(logger.Info),
	})
	// 如果有错误进行返回, 并进行打印报错
	if err != nil {
		fmt.Println("Connect Mysql Server Error:", err)
		return
	}

	db, err := Db.DB()
	if err != nil {
		fmt.Println("sql DB init Error:", err)
		return
	}
	// db连接测试连通性方法
	err = db.Ping()
	if err != nil {
		fmt.Println("Db ping Error:", err)
		return
	}
	fmt.Println("Connect Mysql SuccessFuly!")
	// 设置db的最大连接数
	db.SetMaxOpenConns(20)
	// 设置db的空闲连接数
	db.SetMaxIdleConns(10)
}

func main() {
	initDb()
}
```

# 11. 结合数据库进行校验BasicAuth 验证

database 代码, 单独建立目录 databases

```go
package databases

import (
	"fmt"

	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

var dbtl *gorm.DB

func init() {
	host := "127.0.0.1"
	port := 3306
	username := "root"
	password := "123456"
	database := "test"
	dsn := fmt.Sprintf("%s:%s@tcp(%s:%d)/%s?charset=utf8&&parseTime=True&loc=Local", username, password, host, port, database)
	var err error
	dbtl, err = gorm.Open(mysql.Open(dsn), &gorm.Config{
		Logger: logger.Default.LogMode(logger.Info),
	})

	if err != nil {
		fmt.Println("Connect Mysql Server Error:", err)
		return
	}

	db, err := dbtl.DB()
	if err != nil {
		fmt.Println("SQL Db Error :", err)
	}
	db.SetMaxOpenConns(200)
	db.SetMaxIdleConns(10)
}

func GetDb() *gorm.DB {
	return dbtl
}
```

main.go

```go
package main

import (
	"fmt"
	"gin_example/databases"
	"net/http"

	"github.com/gin-gonic/gin"
	"gorm.io/gorm"
)

// 定义数据库模型
type User struct {
	gorm.Model
	Username string `gorm:"cloumn:username"`
	Password string `gorm:"cloumn:password"`
}

// 数据库表名
func (user User) TableName() string {
	return "user"
}

// 基础用户信息
var userss gin.H = gin.H{
	"nick": gin.H{
		"id":   1,
		"name": "nick",
		"age":  13,
	},
	"tom": gin.H{
		"id":   2,
		"name": "tom",
		"age":  13,
	},
	"jerry": gin.H{
		"id":   2,
		"name": "jerry",
		"age":  13,
	},
}

func UserLoginAccounts() gin.Accounts {
	// 声明users User用户结构体
	var users []User
	// 定义一个默认值
	var accounts = gin.Accounts{
		"default": "123456",
	}
	// 初始化数据库链接完毕
	db := databases.GetDb()
	// 进行查询操作
	result := db.Find(&users)
	if result.Error != nil {
		fmt.Println("Ret gin.Accounts:", result.Error)
		return accounts
	}

	// 加入到accounts里面
	for _, user := range users {
		accounts[user.Username] = user.Password
	}
	return accounts

}

// 进行用户验证的结构体
func UserHandler(c *gin.Context) {
	username := c.MustGet(gin.AuthUserKey).(string)
	if info, ok := userss[username]; ok {
		c.JSON(http.StatusOK, gin.H{
			"profile": info,
		})
	}
}

// 主页欢迎页面
func WelComeHandler(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{
		"msg": "welcome to user basic auth html!",
	})
}

func main() {
	r := gin.Default()
	// 欢迎页面路由
	r.GET("/", WelComeHandler)
	// /v1路由的都会进行基础验证
	g := r.Group("/v1", gin.BasicAuth(UserLoginAccounts()))
	g.GET("/", UserHandler)
	r.Run()
}
```

:测试结果:

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%2017.png)

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%2018.png)