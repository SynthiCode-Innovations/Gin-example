# Golang Gin Web Framework

Created: April 13, 2023 10:25 PM
Reviewed: No

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

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled.png)

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

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%201.png)

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

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%202.png)

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

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%203.png)

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%204.png)

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

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%205.png)

提交请求/toregister

![Untitled](Golang%20Gin%20Web%20Framework%20dfe287992ca446ee8d11756ea17d9d56/Untitled%206.png)