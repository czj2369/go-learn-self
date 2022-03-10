# GO

GO版本：1.17.5

GOLand版本：goland-2021.1.1.win

官方文档地址：http://docscn.studygolang.com/doc/

## 1.下载和安装

下载地址：https://go.dev/dl/

这里使用windows的环境

下载完后安装，接着添加环境变量

![image-20211220211723590](/images/image-20211220211723590.png)



变量值为GO安装的目录

打开CMD验证是否安装成功

```
go version
```

![image-20211220212114022](/images/image-20211220212114022.png)



Linux环境

```
1.Extract the archive you downloaded into /usr/local, creating a Go tree in /usr/local/go.
Important: This step will remove a previous installation at /usr/local/go, if any, prior to extracting. Please back up any data before proceeding.

For example, run the following as root or through sudo:

rm -rf /usr/local/go && tar -C /usr/local -xzf go1.17.5.linux-amd64.tar.gz
2.Add /usr/local/go/bin to the PATH environment variable.
You can do this by adding the following line to your $HOME/.profile or /etc/profile (for a system-wide installation):

export PATH=$PATH:/usr/local/go/bin
Note: Changes made to a profile file may not apply until the next time you log into your computer. To apply the changes immediately, just run the shell commands directly or execute them from the profile using a command such as source $HOME/.profile.

3.Verify that you've installed Go by opening a command prompt and typing the following command:
$ go version
4.Confirm that the command prints the installed version of Go.
```



## 2.编写一个"Hello World!"

### 1.创建一个目录

### 2.在目录中为你的代码启用dependency tracking(依赖跟踪)

通过使用

```
go mod init [module-path]
```

创建一个go.mod文件，这个文件会记录你使用的包的模块。大多数情况下[module-path]使用的是你的仓库地址，例如github.com/czj，如果你想要发布你的模块给其他人使用，那么模块路径一定要GO TOOLS可以下载到的地址。

参考资料：[`go mod init` command](http://docscn.studygolang.com/ref/mod#go-mod-init)

### 3.创建一个文件hello.go

### 4.输入以下代码

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

```go
package main 声明一个main package
package是function的集合，也是由相同目录下的所有文件组成
```

```go
import "fmt" 导入一个名为“fmt”的模块，提供格式化文本，包括打印到控制台等功能，这是一个标准的库包，GO自带，类似于Java中的java.lang
```

```GO
func main(){} 创建一个main方法，当你执行main package的时候，会默认执行这个方法，类似Java的public static void main(String[] args){}
```

```go
fmt.Println("Hello, World!") 向控制台打印Hello World
```

参考资料： [`fmt` package](https://pkg.go.dev/fmt/)

### 5.执行程序

在hello.go目录下执行命令

```go
go run .
```

![image-20211220220210255](/images/image-20211220220210255.png)

参考资料：[`go run` command](http://docscn.studygolang.com/cmd/go/#hdr-Compile_and_run_Go_program)

### 6.使用外部包

有时候你需要使用一些网络上的外部包，可以这样去使用

浏览网站 [pkg.go.dev](pkg.go.dev) 查询你需要引用的包，比如这里查询“[quote](https://pkg.go.dev/search?q=quote)”这个包，模块路劲为 `rsc.io/quote`的包，可以点进详情界面，查看那些function是你需要的

在刚刚的hello.go中添加并修改以下代码使用

```go
package main

import "fmt"

import "rsc.io/quote"

func main() {
    fmt.Println(quote.Go())
}
```

接着输入以下命令将导入的`rsc.io/quote`模块下载

```
go mod tidy
```

![image-20211220221838609](/images/image-20211220221838609.png)

如果遇到无法下载的情况，可以使用以下命令解决

![image-20211220221914116](/images/image-20211220221914116.png)

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```

运行程序

![image-20211220222001117](/images/image-20211220222001117.png)

此时打开go.mod文件，会发现新增内容

```
module example.com/godemo

go 1.17

require rsc.io/quote v1.5.2

require (
	golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c // indirect
	rsc.io/sampler v1.3.0 // indirect
)
```

## 3.创建一个GO模块

1.创建一个greetings模块

```go
go mod init example.com/greetings
```

2.创建一个go文件，输入以下代码

```go
package greetings

import "fmt"

// Hello 方法返回一个name的问候
// Hello 方法的入参是一个字符串，参数名为name，返回一个字符串
func Hello(name string) string {
    // 返回一个带有name的问候
    // message := 等价于 var message string
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message
}
```

## 4.使用你的模块

1.创建一个hello模块

```
go mod init example.com/greetings
```

2.创建一个go文件，输入以下代码

```go
package main

import (
	"example.com/greetings"
	"fmt"
)

func main() {
	message := greetings.Hello("czj")
	fmt.Println(message)
}
```

因为你的模块是基于本地的，go tools 无法下载到你的模块，需要进行以下操作

在hello目录执行以下命令

```
go mod edit -replace example.com/greetings=../greetings
```

这个命令表示将 example.com/greetings 被替换成目录 ../greetings

在go.mod文件中会发现多了一行代码

```
module example.com/hello

go 1.17

replace example.com/greetings => ../greetings
```

此时再执行

```
go tidy
```

即可获得你的模块，此时在go.mod文件中会再多一行代码

```
module example.com/hello

go 1.17

replace example.com/greetings => ../greetings

require example.com/greetings v0.0.0-00010101000000-000000000000
```

跟在模块名后面的字符串是一段伪版本数字，未发布的版本只是用一串随机生成的版本号

参考资料：[Module version numbering](http://docscn.studygolang.com/doc/modules/version-numbers).

					[`require` directive](http://docscn.studygolang.com/doc/modules/gomod-ref#require)

## 5.返回并处理一个错误

使用标准库中的errors包，使用errors.New()方法可以返回一个错误

例如：

在greetings.go中

```Go
package greetings

import (
    "errors"
    "fmt"
)

// Hello 返回对name的问候
func Hello(name string) (string, error) {
    // 如果name为空 返回“name为空”的错误提示
    if name == "" {
        return "", errors.New("name为空")
    }

    // 如果name不为空，则返回nil，表示没有错误
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message, nil
}
```

在hello.go中

```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // 配置log日志打印的开头
    log.SetPrefix("greetings: ")
    // 设置时间标识
    // 0 不带时间 
    // 1 yyyy-MM-dd 
    // 2 HH:mm:SS
    // 3 yyyy-MM-dd HH:mm:SS
    log.SetFlags(0)

    // 调用greetings.go的Hello方法，这里传入空的字符串，测试是否能得到错误提示
    message, err := greetings.Hello("")
    // Fatal 等价于 Print() 然后调用 os.Exit(1)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println(message)
}
```

参考资料：[`log` package](https://pkg.go.dev/log/)

					[`Fatal` function](https://pkg.go.dev/log?tab=doc#Fatal)

## 6.使用随机函数

修改greetings.go文件

```go
package greetings

import (
	"errors"
	"fmt"
	"math/rand"
	"time"
)

func Hello(name string) (string, error) {
	// 对name进行判断
	if name == "" {
		return "", errors.New("name为空")
	}
	message := fmt.Sprintf(randomFormat(), name)
	return message, nil
}
// 初始化函数，用于初始化变量
func init() {
	rand.Seed(time.Now().UnixNano())
}
// 一个随机的问候
func randomFormat() string {
	formats := []string{
		"Hi, %v. Welcome!",
		"Great to see you, %v!",
		"Hail, %v! Well met!",
	}

	return formats[rand.Intn(len(formats))]
}
```

使用了两个新的package，"math/rand"和"time"，一个是随机数，另一个是获取时间

另外使用了一个GO slice，数组类型，类似于array，当你增加或删除数据时，它会动态改变大小

再另外，这里有一个名为init的函数，它是用于初始化数据

参考资料：[`math/rand` package](https://pkg.go.dev/math/rand/)

					[init Effective Go](http://docscn.studygolang.com/doc/effective_go.html#init)

## 7.使用数组参数

修改greeting.go文件

```go
package greetings

import (
	"errors"
	"fmt"
	"math/rand"
	"time"
)

func Hello(name string) (string, error) {
	// 对name进行判断
	if name == "" {
		return "", errors.New("name为空")
	}
	message := fmt.Sprintf(randomFormat(), name)
	return message, nil
}

// 入参为数组，返回一个map,key为string，value为string
func Hellos(names []string)(map[string]string, error) {
	// make(map[key-type]value-type) 初始化map的语法
	messages := make(map[string]string)

	// 循环用法，如果需要当前数组的索引，则将"_"替换成其它，比如index，不需要下标就下划线
	for _, name := range names {
		message, err := Hello(name)
		if err != nil {
			return nil, err
		}
		messages[name] = message
	}
	return messages, nil
}

// 初始化函数，用于初始化变量
func init() {
	rand.Seed(time.Now().UnixNano())
}
// 一个随机的问候
func randomFormat() string {
	formats := []string{
		"Hi, %v. Welcome!",
		"Great to see you, %v!",
		"Hail, %v! Well met!",
	}

	return formats[rand.Intn(len(formats))]
}

```

注意，添加了一个Hellos函数，并且将入参修改成数组，返回修改成map，以及在函数体中使用了循环的逻辑

修改hello.go文件

```go
package main

import (
	"example.com/greetings"
	"fmt"
	"log"
)

func main() {

	log.SetPrefix("greetings:")
	log.SetFlags(0)

	names := []string{"czj", "czj111", "czj222"}

	message, err := greetings.Hellos(names)

	if err != nil {
		log.Fatal(err)
	}

	fmt.Println(message)
}
```

添加一个names变量，使用方法Hellos

参考资料： [Go maps in action](https://docs.studygolang.com/blog/maps)

					[The blank identifier](http://docscn.studygolang.com/doc/effective_go.html#blank)

## 8.单元测试

在greetings文件夹中创建greetings_test.go文件，_test.go为后缀，go test会识别出文件中的test方法

输入以下代码

```go
package greetings

import (
	"regexp"
	"testing"
)

func TestHelloName(t *testing.T) {
	name := "czj"
	want := regexp.MustCompile(`\b` + name + `\b`)
	msg, err := Hello("czj")
	if !want.MatchString(msg) || err != nil {
		t.Fatalf(`Hello("czj") = %q, %v, want match for %#q, nil`, msg, err, want)
	}
}

func TestHelloEmpty(t *testing.T) {
	msg, err := Hello("")
	if msg != "" || err == nil{
		t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
	}
}
```

通过命令 go test或者go test -v（获得详细的输出）可以执行该文件，结果如下

![image-20211226215137324](/images/image-20211226215137324.png)

参考资料：[`Fatalf` method](https://pkg.go.dev/testing/#T.Fatalf)

					[`go test` command](http://docscn.studygolang.com/cmd/go/#hdr-Test_packages)

## 9.编译和安装

通过命令[`go build` command](http://docscn.studygolang.com/cmd/go/#hdr-Compile_packages_and_dependencies)进行编译

通过命令[`go install` command](http://docscn.studygolang.com/ref/mod#go-install)进行安装

参考资料：http://docscn.studygolang.com/doc/tutorial/compile-install

## 10.用GO和GIN开发一个RESTful API接口

### 1.学习GIN

gin是一个用go编写的web框架

### 2.快速开始

使用命令

```
go get -u github.com/gin-gonic/gin
```

进行安装，安装成功之后即可在代码中使用

新建一个模块 web-service-gin，go mod init example.com/web-service-gin

在web-service-gin下新建一个main.go文件，输入以下代码

```GO
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

// 创建一个类型结构，这是使用“专辑”来作为例子
type album struct {
	ID     string  `json:"id"`
	Title  string  `json:"title"`
	Artist string  `json:"artist"`
	Price  float64 `json:"price"`
}

var albums = []album{
	{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
	{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
	{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}

// 编写一个方法来响应返回所有items
func getAlbums(c *gin.Context) {// gin.Context带着请求详细信息、验证和序列化 JSON 等
	// 将结构序列化为 JSON 并将其添加到响应中
	c.IndentedJSON(http.StatusOK, albums)
}

func main() {
	router := gin.Default()
	router.GET("/albums", getAlbums)

	router.Run("localhost:8090")
}
```

tip：如果有报红情况，使用go mod tidy命令

启动项目

```
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /albums                   --> main.getAlbums (3 handlers)
[GIN-debug] [WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.
Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.
[GIN-debug] Listening and serving HTTP on localhost:8090
```

控制台出现以上提示说明启动成功，监听本地8080端口

打开postman，访问http://127.0.0.1:8090/albums，有正常数据返回

![image-20211227001008625](/images/image-20211227001008625.png)

参考资料：[`gin.Context`](https://pkg.go.dev/github.com/gin-gonic/gin#Context)

					[`Context.IndentedJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.IndentedJSON)
	
					[`StatusOK`](https://pkg.go.dev/net/http#StatusOK)
	
					[`Context.JSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.JSON)

### 3.编写通过id查询专辑

完整代码

```go
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

// 创建一个类型结构，这是使用“专辑”来作为例子
type album struct {
	ID     string  `json:"id"`
	Title  string  `json:"title"`
	Artist string  `json:"artist"`
	Price  float64 `json:"price"`
}

var albums = []album{
	{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
	{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
	{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}

// 编写一个方法来响应返回所有items
func getAlbums(c *gin.Context) {// gin.Context带着请求详细信息、验证和序列化 JSON 等
	// 将结构序列化为 JSON 并将其添加到响应中
	c.IndentedJSON(http.StatusOK, albums)
}

func postAlbums(c *gin.Context) {
	var newAlbum album

	// 调用 BindJSON 将接收到的 JSON 绑定到newAlbum
	if err := c.BindJSON(&newAlbum); err != nil {
		return
	}

	// 将新专辑添加到 slice.
	albums = append(albums, newAlbum)
	c.IndentedJSON(http.StatusCreated, newAlbum)
}

// 通过id查询专辑
func getAlbumByID(c *gin.Context) {
	// 获取参数id的值
	id := c.Param("id")

	// 找到目标专辑
	for _, a := range albums {
		if a.ID == id {
			c.IndentedJSON(http.StatusOK, a)
			return
		}
	}
	c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}

func main() {
	// 使用默认方法初始化gin
	router := gin.Default()
	// 使用 GET/POST 函数将 GET/POST HTTP 方法和 /albums 路径与处理程序函数相关联
	router.GET("/albums", getAlbums)
	router.POST("/albums", postAlbums)
    // 在 Gin 中，路径中项目前面的冒号表示路径参数
	router.GET("/getAlbumByID/:id", getAlbumByID)
	router.Run("localhost:8090")
}
```

新增方法getAlbumByID，在main中新增get关联，router.GET("/getAlbumByID/:id", getAlbumByID)

请求地址http://127.0.0.1:8090/getAlbumByID/1即可得到结果

## 11.编写一个Web Application

## 12.数据库连接

创建一个新模块data-access

浏览[SQLDrivers](https://github.com/golang/go/wiki/SQLDrivers)找到合适的数据库驱动，这里使用的数据库是[Postgres](https://github.com/lib/pq)

## 13.标准的GO项目布局

在java中，一般的springBoot项目布局如下列情况

![image-20220104223749040](/images/image-20220104223749040.png)

这样便于我们去很清晰的去了解项目，同样的，GO也有它自己的项目布局

### **Go 目录**

### /cmd

本项目的主干。

每个应用程序的目录名应该与你想要的可执行文件的名称相匹配(例如，`/cmd/myapp`)。

不要在这个目录中放置太多代码。如果你认为代码可以导入并在其他项目中使用，那么它应该位于 `/pkg` 目录中。如果代码不是可重用的，或者你不希望其他人重用它，请将该代码放到 `/internal` 目录中。你会惊讶于别人会怎么做，所以要明确你的意图!

通常有一个小的 `main` 函数，从 `/internal` 和 `/pkg` 目录导入和调用代码，除此之外没有别的东西。

有关示例，请参阅 [`/cmd`](https://github.com/golang-standards/project-layout/blob/master/cmd/README.md) 目录。

项目中的所有你将要编译成可执行程序的入口代码都放在`cmd/` 文件夹里，这些代码和业务没有关系。每个程序对应一个文件夹，文件夹的名称应该以程序的名称命名。一般在名称后面加上`d` 代表该程序是一个守护进程运行。 每个文件夹必须有一个`main`包的源文件，该源文件的名称也最好命名成可执行程序的名称，当然也可以保留main文件名。在此会导入和调用`internal/`和`pkg/`等其他文件夹中相关的代码。

示例

```
├── cmd/
│   └── eventtimer/
│       └── update/
│       └── main.go
│   └── eventserver/
│       └── router/
│           └── handler/
│           └── router.go
│       └── tests/
│       └── main.go
```

- 该项目包含线上业务服务eventserver（提供restful API）、定时器eventtimer（定时更新数据的状态）二个应用程序。`cmd`文件夹对应有2个文件夹，并且每个文件夹下面都有一个`main`包的源文件，至于名称可以直接用main，也可以对应文件夹的名称。
- 每个文件夹下的源文件里的代码和业务逻辑基本没任何关系。比如rest ful的eventserver，里面仅包含router的配置和相关的handler。

### /internal

私有应用程序和库代码。这是你不希望其他人在其应用程序或库中导入代码。请注意，这个布局模式是由 Go 编译器本身执行的。有关更多细节，请参阅Go 1.4 [`release notes`](https://golang.org/doc/go1.4#internalpackages) 。注意，你并不局限于顶级 `internal` 目录。在项目树的任何级别上都可以有多个内部目录。

你可以选择向 internal 包中添加一些额外的结构，以分隔共享和非共享的内部代码。这不是必需的(特别是对于较小的项目)，但是最好有有可视化的线索来显示预期的包的用途。你的实际应用程序代码可以放在 `/internal/app` 目录下(例如 `/internal/app/myapp`)，这些应用程序共享的代码可以放在 `/internal/pkg` 目录下(例如 `/internal/pkg/myprivlib`)。



在go语言中，变量，函数，方法等的存取权限只有exported(全局)和unexported(包可见，局部)2种。

在项目中不被复用，也不能被其他项目导入，仅被本项目内部使用的代码包即私有的代码包都应该放在`internal`文件夹下。该文件夹下的所有包及相应文件都有一个项目保护级别，即其他项目是不能导入这些包的，仅仅是该项目内部使用。

如果你在其他项目中导入另一个项目的`internal`的代码包，保存或`go build` 编译时会报错`use of internal package ... not allowed`，该特性是在go 1.4版本开始支持的，编译时强行校验。

```
1 package main
2
3 import (
4	"paper-code/examples/groupevent/cmd/eventserver/router/handler"
5	"paper-code/examples/groupevent/cmd/internal"
6	"paper-code/examples/groupevent/internal/eventpopdserver/event"
7	"paper-code/examples/groupevent/pkg/middleware"
8 )
9
10 func main() {
11 	middleware.HandlerConv(nil)
12
13	event.EventsBy("")
14
15	eh := new(handler.EventHandler)
16	eh.Events(nil, nil)
17
18	internal.CmdInternalFunc()
19 }
```

> 此代码片段为另一个项目导入paper-code/example/groupevent的代码包

> 第6行的导入就会提示`use of internal package paper-code/examples/groupevent/internal/eventpopdserver/event not allowed`

> 第5行的导入也会提示同样的错误

> 第7行导入就可以的，因为导入的pkg代码包

当然你也不要局限根目录下的`internal`目录，你也可以在任何一个目录中创建`internal`，规则都适用。比如上面的例子`第5行的导入也会提示同样的错误:use of internal package paper-code/examples/groupevent/cmd/internal not allowed`

另外在同一个项目中，`internal`包的导入规则是：`.../a/b/c/internal/d/e/f` 仅仅可以被`.../a/b/c`下的目录导入，`.../a/b/g`则不允许。

你可以在`internal`文件夹添加其他的架构分层目录来区分可分享、不可分享的代码，比如`internal/myapp`是你项目中某个程序的不可分享的代码；`internal/pkg/`是你项目中的程序都可以分享的代码。也可以添加数据层、业务逻辑层的代码，这个属于在项目中更通用的一个架构分层，和这里的包设计并不冲突，即上层模块可以直接访问下层模块，反之不然。

#### internal/pkg/

在同一个项目中不同程序需要访问，但又不能让其他项目访问的代码包，需要放在这里。这些包是比较基础但又提供了很特殊的功能，比如数据库、日志、用户验证等功能。

### /pkg

外部应用程序可以使用的库代码(例如 `/pkg/mypubliclib`)。其他项目会导入这些库，希望它们能正常工作，所以在这里放东西之前要三思:-)注意，`internal` 目录是确保私有包不可导入的更好方法，因为它是由 Go 强制执行的。`/pkg` 目录仍然是一种很好的方式，可以显式地表示该目录中的代码对于其他人来说是安全使用的好方法。由 Travis Jeffery 撰写的 [`I'll take pkg over internal`](https://travisjeffery.com/b/2019/11/i-ll-take-pkg-over-internal/) 博客文章提供了 `pkg` 和 `internal` 目录的一个很好的概述，以及什么时候使用它们是有意义的。

当根目录包含大量非 Go 组件和目录时，这也是一种将 Go 代码分组到一个位置的方法，这使得运行各种 Go 工具变得更加容易（正如在这些演讲中提到的那样: 来自 GopherCon EU 2018 的 [`Best Practices for Industrial Programming`](https://www.youtube.com/watch?v=PTE4VJIdHPg) , [GopherCon 2018: Kat Zien - How Do You Structure Your Go Apps](https://www.youtube.com/watch?v=oL6JBUk6tj0) 和 [GoLab 2018 - Massimiliano Pippi - Project layout patterns in Go](https://www.youtube.com/watch?v=3gQa1LWwuzk) ）。

如果你想查看哪个流行的 Go 存储库使用此项目布局模式，请查看 [`/pkg`](https://github.com/golang-standards/project-layout/blob/master/pkg/README.md) 目录。这是一种常见的布局模式，但并不是所有人都接受它，一些 Go 社区的人也不推荐它。

如果你的应用程序项目真的很小，并且额外的嵌套并不能增加多少价值(除非你真的想要:-)，那就不要使用它。当它变得足够大时，你的根目录会变得非常繁琐时(尤其是当你有很多非 Go 应用组件时)，请考虑一下。

### /vendor

应用程序依赖项(手动管理或使用你喜欢的依赖项管理工具，如新的内置 [`Go Modules`](https://github.com/golang/go/wiki/Modules) 功能)。`go mod vendor` 命令将为你创建 `/vendor` 目录。请注意，如果未使用默认情况下处于启用状态的 Go 1.14，则可能需要在 `go build` 命令中添加 `-mod=vendor` 标志。

如果你正在构建一个库，那么不要提交你的应用程序依赖项。

注意，自从 [`1.13`](https://golang.org/doc/go1.13#modules) 以后，Go 还启用了模块代理功能(默认使用 [`https://proxy.golang.org`](https://proxy.golang.org/) 作为他们的模块代理服务器)。在[`here`](https://blog.golang.org/module-mirror-launch) 阅读更多关于它的信息，看看它是否符合你的所有需求和约束。如果需要，那么你根本不需要 `vendor` 目录。

国内模块代理功能默认是被墙的，七牛云有维护专门的的[`模块代理`](https://github.com/goproxy/goproxy.cn/blob/master/README.zh-CN.md) 。

### 服务应用程序目录

### /api

OpenAPI/Swagger 规范，JSON 模式文件，协议定义文件。

有关示例，请参见 [`/api`](https://github.com/golang-standards/project-layout/blob/master/api/README.md) 目录。

### Web 应用程序目录

### /web

特定于 Web 应用程序的组件:静态 Web 资产、服务器端模板和 SPAs。

### 通用应用目录

### /configs

配置文件模板或默认配置。

将你的 `confd` 或 `consul-template` 模板文件放在这里。

### /init

System init（systemd，upstart，sysv）和 process manager/supervisor（runit，supervisor）配置。

### /scripts

执行各种构建、安装、分析等操作的脚本。

这些脚本保持了根级别的 Makefile 变得小而简单(例如， [`https://github.com/hashicorp/terraform/blob/master/Makefile`](https://github.com/hashicorp/terraform/blob/master/Makefile) )。

有关示例，请参见  [`/scripts`](https://github.com/golang-standards/project-layout/blob/master/scripts/README.md) 目录。

### /build

打包和持续集成。

将你的云( AMI )、容器( Docker )、操作系统( deb、rpm、pkg )包配置和脚本放在 `/build/package` 目录下。

将你的 CI (travis、circle、drone)配置和脚本放在 `/build/ci` 目录中。请注意，有些 CI 工具(例如 Travis CI)对配置文件的位置非常挑剔。尝试将配置文件放在 `/build/ci` 目录中，将它们链接到 CI 工具期望它们的位置(如果可能的话)。

### /deployments

IaaS、PaaS、系统和容器编排部署配置和模板(docker-compose、kubernetes/helm、mesos、terraform、bosh)。注意，在一些存储库中(特别是使用 kubernetes 部署的应用程序)，这个目录被称为 `/deploy`。

### /test

额外的外部测试应用程序和测试数据。你可以随时根据需求构造 `/test` 目录。对于较大的项目，有一个数据子目录是有意义的。例如，你可以使用 `/test/data` 或 `/test/testdata` (如果你需要忽略目录中的内容)。请注意，Go 还会忽略以“.”或“_”开头的目录或文件，因此在如何命名测试数据目录方面有更大的灵活性。

有关示例，请参见 [`/test`](https://github.com/golang-standards/project-layout/blob/master/test/README.md) 目录。

### 其他目录

### /docs

设计和用户文档(除了 godoc 生成的文档之外)。

有关示例，请参阅 [`/docs`](https://github.com/golang-standards/project-layout/blob/master/docs/README.md) 目录。

### /tools

这个项目的支持工具。注意，这些工具可以从 `/pkg` 和 `/internal` 目录导入代码。

有关示例，请参见 [`/tools`](https://github.com/golang-standards/project-layout/blob/master/tools/README.md) 目录。

### /examples

你的应用程序和/或公共库的示例。

有关示例，请参见 [`/examples`](https://github.com/golang-standards/project-layout/blob/master/examples/README.md) 目录。

### /third_party

外部辅助工具，分叉代码和其他第三方工具(例如 Swagger UI)。

### /githooks

Git hooks。

### /assets

与存储库一起使用的其他资产(图像、徽标等)。

### /website

如果你不使用 Github 页面，则在这里放置项目的网站数据。

有关示例，请参见 [`/website`](https://github.com/golang-standards/project-layout/blob/master/website/README.md) 目录。

### 你不应该拥有的目录

### /src

有些 Go 项目确实有一个 `src` 文件夹，但这通常发生在开发人员有 Java 背景，在那里它是一种常见的模式。如果可以的话，尽量不要采用这种 Java 模式。你真的不希望你的 Go 代码或 Go 项目看起来像 Java:-)

不要将项目级别 `src` 目录与 Go 用于其工作空间的 `src` 目录(如 [`How to Write Go Code`](https://golang.org/doc/code.html) 中所述)混淆。`$GOPATH` 环境变量指向你的(当前)工作空间(默认情况下，它指向非 windows 系统上的 `$HOME/go`)。这个工作空间包括顶层 `/pkg`, `/bin` 和 `/src` 目录。你的实际项目最终是 `/src` 下的一个子目录，因此，如果你的项目中有 `/src` 目录，那么项目路径将是这样的: `/some/path/to/workspace/src/your_project/src/your_code.go`。注意，在 Go 1.11 中，可以将项目放在 `GOPATH` 之外，但这并不意味着使用这种布局模式是一个好主意。



一个典型的应用项目结构应该是这样的：

```
paper-code/examples/groupevent
├── cmd/
│   └── eventtimer/
│       └── update/
│       └── main.go
│   └── eventserver/
│       └── router/
│           └── handler/
│           └── router.go
│       └── tests/
│       └── main.go
├── internal/
│   └── eventserver/
│       └── biz/
│           └── event/
│           └── member/
│       └── data/
│           └── service/
│   └── eventpopdserver/
│       └── event/
│       └── member/
│   └── pkg/
│       └── cfg/
│       └── db/
│       └── log/
└── vendor/
│   ├── github.com/
│   │   ├── ardanlabs/
│   │   ├── golang/
│   │   ├── prometheus/
│   └── golang.org/
├── go.mod
├── go.sum
```

参考资料：https://github.com/golang-standards/project-layout

https://github.com/danceyoung/paper-code/blob/master/package-oriented-design/packageorienteddesign.md



## 14.GO容器

介绍数组、切片、映射，以及列表的增加、删除、修改和遍历的使用方法

### 1.数组

数组的声明语法如下：

```go
var 数组变量名 [元素数量]Type
```

语法说明如下所示：

- 数组变量名：数组声明及使用时的变量名。
- 元素数量：数组的元素数量，可以是一个表达式，但最终通过编译期计算的结果必须是整型数值，元素数量不能含有到运行时才能确认大小的数值。
- Type：可以是任意基本类型，包括数组本身，类型为数组本身时，可以实现多维数组。

遍历数组也和遍历切片类似，代码如下所示：

```go
var team [3]string
team[0] = "hammer"
team[1] = "soldier"
team[2] = "mum"
for k, v := range team {
    fmt.Println(k, v)
}
```

声明多维数组的语法如下所示：

```go
var array_name [size1][size2]...[sizen] array_type
```

参考资料：http://c.biancheng.net/view/26.html

http://c.biancheng.net/view/4117.html



### 2.切片

切片（slice）是对数组的一个连续片段的引用，所以切片是一个引用类型（因此更类似于 C/[C++](http://c.biancheng.net/cplus/) 中的数组类型，或者 [Python](http://c.biancheng.net/python/) 中的 list 类型），这个片段可以是整个数组，也可以是由起始和终止索引标识的一些项的子集，需要注意的是，终止索引标识的项不包括在切片内。

从连续内存区域生成切片是常见的操作，格式如下：

```
slice [开始位置 : 结束位置]
```

语法说明如下：

- slice：表示目标切片对象；
- 开始位置：对应目标切片对象的索引；
- 结束位置：对应目标切片的结束索引。


从数组生成切片，代码如下：

```
var a  = [3]int{1, 2, 3}
fmt.Println(a, a[1:2])
结果：
[1 2 3]  [2]
```

从数组或切片生成新的切片拥有如下特性：

- 取出的元素数量为：结束位置 - 开始位置；
- 取出元素不包含结束位置对应的索引，切片最后一个元素使用 slice[len(slice)] 获取；
- 当缺省开始位置时，表示从连续区域开头到结束位置；
- 当缺省结束位置时，表示从开始位置到整个连续区域末尾；
- 两者同时缺省时，与切片本身等效；
- 两者同时为 0 时，等效于空切片，一般用于切片复位。

除了可以从原有的数组或者切片中生成切片外，也可以声明一个新的切片，每一种类型都可以拥有其切片类型，表示多个相同类型元素的连续集合，因此切片类型也可以被声明，切片类型声明格式如下：

```go
var name []Type
```

如果需要动态地创建一个切片，可以使用 make() 内建函数，格式如下：

```go
make( []Type, size, cap )
```

其中 Type 是指切片的元素类型，size 指的是为这个类型分配多少个元素，cap 为预分配的元素数量，这个值设定后不影响 size，只是能提前分配空间，降低多次分配空间造成的性能问题。 

使用 make() 函数生成的切片一定发生了内存分配操作，但给定开始与结束位置（包括切片复位）的切片只是将新的切片结构指向已经分配好的内存区域，设定开始与结束位置，不会发生内存分配操作。

#### 添加元素

Go语言的内建函数 append() 可以为切片动态添加元素，代码如下所示：

```go
var a []int
a = append(a, 1) // 追加1个元素
a = append(a, 1, 2, 3) // 追加多个元素, 手写解包方式
a = append(a, []int{1,2,3}...) // 追加一个切片, 切片需要解包
```

在使用 append() 函数为切片动态添加元素时，如果空间不足以容纳足够多的元素，切片就会进行“扩容”，此时新切片的长度会发生改变。切片在扩容时，容量的扩展规律是按容量的 2 倍数进行扩充，例如 1、2、4、8、16……

除了在切片的尾部追加，我们还可以在切片的开头添加元素：

```go
var a = []int{1,2,3}
a = append([]int{0}, a...) // 在开头添加1个元素
a = append([]int{-3,-2,-1}, a...) // 在开头添加1个切片
```

在切片开头添加元素一般都会导致内存的重新分配，而且会导致已有元素全部被复制 1 次，因此，从切片的开头添加元素的性能要比从尾部追加元素的性能差很多。

因为 append 函数返回新切片的特性，所以切片也支持链式操作，我们可以将多个 append 操作组合起来，实现在切片中间插入元素：

```go
var a []int
a = append(a[:i], append([]int{x}, a[i:]...)...) // 在第i个位置插入x
a = append(a[:i], append([]int{1,2,3}, a[i:]...)...) // 在第i个位置插入切片
```

每个添加操作中的第二个 append 调用都会创建一个临时切片，并将 a[i:] 的内容复制到新创建的切片中，然后将临时创建的切片再追加到 a[:i] 中。

#### 切片复制

Go语言的内置函数 copy() 可以将一个数组切片复制到另一个数组切片中，如果加入的两个数组切片不一样大，就会按照其中较小的那个数组切片的元素个数进行复制。

copy() 函数的使用格式如下：

```
copy( destSlice, srcSlice []T) int
```

其中 srcSlice 为数据来源切片，destSlice 为复制的目标（也就是将 srcSlice 复制到 destSlice），目标切片必须分配过空间且足够承载复制的元素个数，并且来源和目标的类型必须一致，copy() 函数的返回值表示实际发生复制的元素个数。

#### 删除元素

Go语言并没有对删除切片元素提供专用的语法或者接口，需要使用切片本身的特性来删除元素，根据要删除元素的位置有三种情况，分别是从开头位置删除、从中间位置删除和从尾部删除，其中删除切片尾部的元素速度最快。

##### **从开头位置删除**

删除开头的元素可以直接移动数据指针：

```go
a = []int{1, 2, 3}
a = a[1:] // 删除开头1个元素
a = a[N:] // 删除开头N个元素
```

也可以不移动数据指针，但是将后面的数据向开头移动，可以用 append 原地完成（所谓原地完成是指在原有的切片数据对应的内存区间内完成，不会导致内存空间结构的变化）：

``` go
a = []int{1, 2, 3}
a = append(a[:0], a[1:]...) // 删除开头1个元素
a = append(a[:0], a[N:]...) // 删除开头N个元素
```

还可以用 copy() 函数来删除开头的元素：

```go
a = []int{1, 2, 3}
a = a[:copy(a, a[1:])] // 删除开头1个元素
a = a[:copy(a, a[N:])] // 删除开头N个元素
```

##### 从中间位置删除

对于删除中间的元素，需要对剩余的元素进行一次整体挪动，同样可以用 append 或 copy 原地完成：

```go
a = []int{1, 2, 3, ...}
a = append(a[:i], a[i+1:]...) // 删除中间1个元素
a = append(a[:i], a[i+N:]...) // 删除中间N个元素
a = a[:i+copy(a[i:], a[i+1:])] // 删除中间1个元素
a = a[:i+copy(a[i:], a[i+N:])] // 删除中间N个元素
```

##### 从尾部删除

```go
a = []int{1, 2, 3}
a = a[:len(a)-1] // 删除尾部1个元素
a = a[:len(a)-N] // 删除尾部N个元素
```

连续容器的元素删除无论在任何语言中，都要将删除点前后的元素移动到新的位置，随着元素的增加，这个过程将会变得极为耗时，因此，当业务需要大量、频繁地从一个切片中删除元素时，如果对性能要求较高的话，就需要考虑更换其他的容器了（如双链表等能快速从删除点删除元素）。

#### 迭代

```go
// 创建一个整型切片，并赋值
slice := []int{10, 20, 30, 40}
// 迭代每一个元素，并显示其值
for index, value := range slice {
    fmt.Printf("Index: %d Value: %d\n", index, value)
}
```

需要强调的是，range 返回的是每个元素的副本，而不是直接返回对该元素的引用，因为迭代返回的变量是一个在迭代过程中根据切片依次赋值的新变量，所以 value 的地址总是相同的，要想获取每个元素的地址，需要使用切片变量和索引值

如果不需要索引值，也可以使用下划线`_`来忽略这个值，代码如下所示。

```go
// 创建一个整型切片，并赋值
slice := []int{10, 20, 30, 40}
// 迭代每个元素，并显示其值
for _, value := range slice {
    fmt.Printf("Value: %d\n", value)
}
```

关键字 range 总是会从切片头部开始迭代。如果想对迭代做更多的控制，则可以使用传统的 for 循环，代码如下所示。

```go
// 创建一个整型切片，并赋值
slice := []int{10, 20, 30, 40}
// 从第三个元素开始迭代每个元素
for index := 2; index < len(slice); index++ {
    fmt.Printf("Index: %d Value: %d\n", index, slice[index])
}
```

### 3.map（Go语言映射）

Go语言中 map 是一种特殊的[数据结构](http://c.biancheng.net/data_structure/)，一种元素对（pair）的无序集合，pair 对应一个 key（索引）和一个 value（值），所以这个结构也称为关联数组或字典，这是一种能够快速寻找值的理想结构，给定 key，就可以迅速找到对应的 value。

map 这种数据结构在其他编程语言中也称为字典（[Python](http://c.biancheng.net/python/)）、hash 和 HashTable 等。

#### map 概念

map 是引用类型，可以使用如下方式声明：

```go
var mapname map[keytype]valuetype
```

其中：

- mapname 为 map 的变量名。
- keytype 为键类型。
- valuetype 是键对应的值类型。

在声明的时候不需要知道 map 的长度，因为 map 是可以动态增长的，未初始化的 map 的值是 nil，使用函数 len() 可以获取 map 中 pair 的数目。

```go
package main
import "fmt"
func main() {
    var mapLit map[string]int
    //var mapCreated map[string]float32
    var mapAssigned map[string]int
    mapLit = map[string]int{"one": 1, "two": 2}
    mapCreated := make(map[string]float32)
    mapAssigned = mapLit
    mapCreated["key1"] = 4.5
    mapCreated["key2"] = 3.14159
    mapAssigned["two"] = 3
    fmt.Printf("Map literal at \"one\" is: %d\n", mapLit["one"])
    fmt.Printf("Map created at \"key2\" is: %f\n", mapCreated["key2"])
    fmt.Printf("Map assigned at \"two\" is: %d\n", mapLit["two"])
    fmt.Printf("Map literal at \"ten\" is: %d\n", mapLit["ten"])
}
```

示例中 mapLit 演示了使用`{key1: value1, key2: value2}`的格式来初始化 map ，就像数组和结构体一样。

上面代码中的 mapCreated 的创建方式`mapCreated := make(map[string]float)`等价于`mapCreated := map[string]float{} `。

mapAssigned 是 mapList 的引用，对 mapAssigned 的修改也会影响到 mapLit 的值。

注意：可以使用 make()，但不能使用 new() 来构造 map，如果错误的使用 new() 分配了一个引用对象，会获得一个空引用的指针，相当于声明了一个未初始化的变量并且取了它的地址：

```go
mapCreated := new(map[string]float)
```

接下来当我们调用`mapCreated["key1"] = 4.5`的时候，编译器会报错：

```go
invalid operation: mapCreated["key1"] (index of type *map[string]float).
```

#### map 容量

和数组不同，map 可以根据新增的 key-value 动态的伸缩，因此它不存在固定长度或者最大限制，但是也可以选择标明 map 的初始容量 capacity，格式如下：

```go
make(map[keytype]valuetype, cap)
```

例如：

```go
map2 := make(map[string]float, 100)
```

当 map 增长到容量上限的时候，如果再增加新的 key-value，map 的大小会自动加 1，所以出于性能的考虑，对于大的 map 或者会快速扩张的 map，即使只是大概知道容量，也最好先标明。

#### 用切片作为 map 的值

既然一个 key 只能对应一个 value，而 value 又是一个原始类型，那么如果一个 key 要对应多个值怎么办？例如，当我们要处理 unix 机器上的所有进程，以父进程（pid 为整形）作为 key，所有的子进程（以所有子进程的 pid 组成的切片）作为 value。通过将 value 定义为 []int 类型或者其他类型的切片，就可以优雅的解决这个问题，示例代码如下所示：

```go
mp1 := make(map[int][]int)
mp2 := make(map[int]*[]int)
```

#### 遍历

map 的遍历过程使用 for range 循环完成，代码如下：

```go
scene := make(map[string]int)
scene["route"] = 66
scene["brazil"] = 4
scene["china"] = 960
for k, v := range scene {
    fmt.Println(k, v)
}
```

遍历对于Go语言的很多对象来说都是差不多的，直接使用 for range 语法即可，遍历时，可以同时获得键和值，如只遍历值，将不需要的键使用`_`改为匿名变量形式，可以使用下面的形式：

```go
for _, v := range scene {}
```

只遍历键时，使用下面的形式：

```go
for k := range scene {}
```

无须将值改为匿名变量形式，忽略值即可。

> 注意：遍历输出元素的顺序与填充顺序无关，不能期望 map 在遍历时返回某种期望顺序的结果。

#### 删除和清空

使用 delete() 内建函数从 map 中删除一组键值对，delete() 函数的格式如下：

```go
delete(map, 键)
```

Go语言中并没有为 map 提供任何清空所有元素的函数、方法，清空 map 的唯一办法就是重新 make 一个新的 map，不用担心垃圾回收的效率，Go语言中的并行垃圾回收效率比写一个清空函数要高效的多。

#### sync.Map

Go语言中的 map 在并发情况下，只读是线程安全的，同时读写是线程不安全的。

需要并发读写时，一般的做法是加锁，但这样性能并不高，Go语言在 1.9 版本中提供了一种效率较高的并发安全的 sync.Map，sync.Map 和 map 不同，不是以语言原生形态提供，而是在 sync 包下的特殊结构。

sync.Map 有以下特性：

- 无须初始化，直接声明即可。
- sync.Map 不能使用 map 的方式进行取值和设置等操作，而是使用 sync.Map 的方法进行调用，Store 表示存储，Load 表示获取，Delete 表示删除。
- 使用 Range 配合一个回调函数进行遍历操作，通过回调函数返回内部遍历出来的值，Range 参数中回调函数的返回值在需要继续迭代遍历时，返回 true，终止迭代遍历时，返回 false。

并发安全的 sync.Map 演示代码如下：

```go
package main
import (
      "fmt"
      "sync"
)
func main() {
    var scene sync.Map
    // 将键值对保存到sync.Map
    scene.Store("greece", 97)
    scene.Store("london", 100)
    scene.Store("egypt", 200)
    // 从sync.Map中根据键取值
    fmt.Println(scene.Load("london"))
    // 根据键删除对应的键值对
    scene.Delete("london")
    // 遍历所有sync.Map中的键值对
    scene.Range(func(k, v interface{}) bool {
        fmt.Println("iterate:", k, v)
        return true
    })
}
```

输出：

```
100 true
iterate: egypt 200
iterate: greece 97
```

sync.Map 没有提供获取 map 数量的方法，替代方法是在获取 sync.Map 时遍历自行计算数量，sync.Map 为了保证并发安全有一些性能损失，因此在非并发情况下，使用 map 相比使用 sync.Map 会有更好的性能。

### 4.Go语言list（列表）

#### 初始化列表

list 的初始化有两种方法：分别是使用 New() 函数和 var 关键字声明，两种方法的初始化效果都是一致的。

1) 通过 container/list 包的 New() 函数初始化 list

```
变量名 := list.New()
```

2) 通过 var 关键字声明初始化 list

```
var 变量名 list.List
```

列表与切片和 map 不同的是，列表并没有具体元素类型的限制，因此，列表的元素可以是任意类型，这既带来了便利，也引来一些问题，例如给列表中放入了一个 interface{} 类型的值，取出值后，如果要将 interface{} 转换为其他类型将会发生宕机。

#### 插入元素

双链表支持从队列前方或后方插入元素，分别对应的方法是 PushFront 和 PushBack。

**提示**

这两个方法都会返回一个 *list.Element 结构，如果在以后的使用中需要删除插入的元素，则只能通过 *list.Element 配合 Remove() 方法进行删除，这种方法可以让删除更加效率化，同时也是双链表特性之一。

列表插入元素的方法如下表所示。

| 方  法                                                | 功  能                                            |
| ----------------------------------------------------- | ------------------------------------------------- |
| InsertAfter(v interface {}, mark * Element) * Element | 在 mark 点之后插入元素，mark 点由其他插入函数提供 |
| InsertBefore(v interface {}, mark * Element) *Element | 在 mark 点之前插入元素，mark 点由其他插入函数提供 |
| PushBackList(other *List)                             | 添加 other 列表元素到尾部                         |
| PushFrontList(other *List)                            | 添加 other 列表元素到头部                         |

#### 删除元素

列表插入函数的返回值会提供一个 *list.Element 结构，这个结构记录着列表元素的值以及与其他节点之间的关系等信息，从列表中删除元素时，需要用到这个结构进行快速删除。

```go
package main
import "container/list"
func main() {
    l := list.New()
    // 尾部添加
    l.PushBack("canon")
    // 头部添加
    l.PushFront(67)
    // 尾部添加后保存元素句柄
    element := l.PushBack("fist")
    // 在fist之后添加high
    l.InsertAfter("high", element)
    // 在fist之前添加noon
    l.InsertBefore("noon", element)
    // 使用
    l.Remove(element)
}
```

下表中展示了每次操作后列表的实际元素情况。

| 操作内容                        | 列表元素                    |
| ------------------------------- | --------------------------- |
| l.PushBack("canon")             | canon                       |
| l.PushFront(67)                 | 67, canon                   |
| element := l.PushBack("fist")   | 67, canon, fist             |
| l.InsertAfter("high", element)  | 67, canon, fist, high       |
| l.InsertBefore("noon", element) | 67, canon, noon, fist, high |
| l.Remove(element)               | 67, canon, noon, high       |

#### 遍历列表——访问列表的每一个元素

遍历双链表需要配合 Front() 函数获取头元素，遍历时只要元素不为空就可以继续进行，每一次遍历都会调用元素的 Next() 函数，代码如下所示。

```go
l := list.New()
// 尾部添加
l.PushBack("canon")
// 头部添加
l.PushFront(67)
for i := l.Front(); i != nil; i = i.Next() {
    fmt.Println(i.Value)
}
```

### 5.Go语言nil：空值/零值

在Go语言中，布尔类型的零值（初始值）为 false，数值类型的零值为 0，字符串类型的零值为空字符串`""`，而指针、切片、映射、通道、函数和接口的零值则是 nil。

nil 是Go语言中一个预定义好的标识符，有过其他编程语言开发经验的开发者也许会把 nil 看作其他语言中的 null（NULL），其实这并不是完全正确的，因为Go语言中的 nil 和其他语言中的 null 有很多不同点。

下面通过几个方面来介绍一下Go语言中 nil。

#### nil 标识符是不能比较的

```go
package main
import (
    "fmt"
)
func main() {
    fmt.Println(nil==nil)
}
```

运行结果如下所示：

```
# command-line-arguments
.\main.go:8:21: invalid operation: nil == nil (operator == not defined on nil)
```

从上面的运行结果不难看出，`==`对于 nil 来说是一种未定义的操作。

#### nil 不是关键字或保留字

nil 并不是Go语言的关键字或者保留字，也就是说我们可以定义一个名称为 nil 的变量，比如下面这样：

```
var nil = errors.New("my god")
```

虽然上面的声明语句可以通过编译，但是并不提倡这么做。

#### nil 没有默认类型

```go
package main
import (
    "fmt"
)
func main() {
    fmt.Printf("%T", nil)
    print(nil)
}
```

结果

```
# command-line-arguments
.\main.go:9:10: use of untyped nil
```

#### 不同类型 nil 的指针是一样的

```go
package main
import (
    "fmt"
)
func main() {
    var arr []int
    var num *int
    fmt.Printf("%p\n", arr)
    fmt.Printf("%p", num)
}
```

```
0x0
0x0
```

通过运行结果可以看出 arr 和 num 的指针都是 0x0。

#### 不同类型的 nil 是不能比较的

```go
package main
import (
    "fmt"
)
func main() {
    var m map[int]string
    var ptr *int
    fmt.Printf(m == ptr)
}
```

```
# command-line-arguments
.\main.go:10:20: invalid operation: arr == ptr (mismatched types []int and *int)
```

#### 两个相同类型的 nil 值也可能无法比较

```go
package main
import (
    "fmt"
)
func main() {
    var s1 []int
    var s2 []int
    fmt.Printf(s1 == s2)
}
```

```
# command-line-arguments
.\main.go:10:19: invalid operation: s1 == s2 (slice can only be compared to nil)
```

通过上面的错误提示可以看出，能够将上述不可比较类型的空值直接与 nil 标识符进行比较，如下所示：

```go
package main
import (
    "fmt"
)
func main() {
    var s1 []int
    fmt.Println(s1 == nil)
}
```

```
true
```

#### nil 是 map、slice、pointer、channel、func、interface 的零值

```go
package main
import (
    "fmt"
)
func main() {
    var m map[int]string
    var ptr *int
    var c chan int
    var sl []int
    var f func()
    var i interface{}
    fmt.Printf("%#v\n", m)
    fmt.Printf("%#v\n", ptr)
    fmt.Printf("%#v\n", c)
    fmt.Printf("%#v\n", sl)
    fmt.Printf("%#v\n", f)
    fmt.Printf("%#v\n", i)
}
```

```
map[int]string(nil)
(*int)(nil)
(chan int)(nil)
[]int(nil)
(func())(nil)
<nil>
```

零值是Go语言中变量在声明之后但是未初始化被赋予的该类型的一个默认值。

#### 不同类型的 nil 值占用的内存大小可能是不一样的

一个类型的所有的值的内存布局都是一样的，nil 也不例外，nil 的大小与同类型中的非 nil 类型的大小是一样的。但是不同类型的 nil 值的大小可能不同。

```go
package main
import (
    "fmt"
    "unsafe"
)
func main() {
    var p *struct{}
    fmt.Println( unsafe.Sizeof( p ) ) // 8
    var s []int
    fmt.Println( unsafe.Sizeof( s ) ) // 24
    var m map[int]bool
    fmt.Println( unsafe.Sizeof( m ) ) // 8
    var c chan string
    fmt.Println( unsafe.Sizeof( c ) ) // 8
    var f func()
    fmt.Println( unsafe.Sizeof( f ) ) // 8
    var i interface{}
    fmt.Println( unsafe.Sizeof( i ) ) // 16
}
```

```
8
24
8
8
8
16
```

具体的大小取决于编译器和架构，上面打印的结果是在 64 位架构和标准编译器下完成的，对应 32 位的架构的，打印的大小将减半。

## 15.Go语言make和new关键字的区别及实现原理

Go语言中 new 和 make 是两个内置函数，主要用来创建并分配类型的内存。在我们定义变量的时候，可能会觉得有点迷惑，不知道应该使用哪个函数来声明变量，其实他们的规则很简单，new 只分配内存，而 make 只能用于 slice、map 和 channel 的初始化，下面我们就来具体介绍一下。

### new

在Go语言中，new 函数描述如下：

```go
// The new built-in function allocates memory. The first argument is a type,
// not a value, and the value returned is a pointer to a newly
// allocated zero value of that type.
func new(Type) *Type
```

从上面的代码可以看出，new 函数只接受一个参数，这个参数是一个类型，并且返回一个指向该类型内存地址的指针。同时 new 函数会把分配的内存置为零，也就是类型的零值。

【示例】使用 new 函数为变量分配内存空间。

```go
var sum *int
sum = new(int) //分配空间
*sum = 98
fmt.Println(*sum)
```

当然，new 函数不仅仅能够为系统默认的数据类型，分配空间，自定义类型也可以使用 new 函数来分配空间，如下所示：

```go
type Student struct {
   name string
   age int
}
var s *Student
s = new(Student) //分配空间
s.name ="dequan"
fmt.Println(s)
```

这里如果我们不使用 new 函数为自定义类型分配空间（将第 7 行注释），就会报错

这就是 new 函数，它返回的永远是类型的指针，指针指向分配类型的内存地址。

### make

make 也是用于内存分配的，但是和 new 不同，它只用于 chan、map 以及 slice 的内存创建，而且它返回的类型就是这三个类型本身，而不是他们的指针类型，因为这三种类型就是引用类型，所以就没有必要返回他们的指针了。

在Go语言中，make 函数的描述如下：

```go
// The make built-in function allocates and initializes an object of type
// slice, map, or chan (only). Like new, the first argument is a type, not a
// value. Unlike new, make's return type is the same as the type of its
// argument, not a pointer to it. The specification of the result depends on
// the type:
// Slice: The size specifies the length. The capacity of the slice is
// equal to its length. A second integer argument may be provided to
// specify a different capacity; it must be no smaller than the
// length, so make([]int, 0, 10) allocates a slice of length 0 and
// capacity 10.
// Map: An empty map is allocated with enough space to hold the
// specified number of elements. The size may be omitted, in which case
// a small starting size is allocated.
// Channel: The channel's buffer is initialized with the specified
// buffer capacity. If zero, or the size is omitted, the channel is
// unbuffered.
func make(t Type, size ...IntegerType) Type
```

通过上面的代码可以看出 make 函数的 t 参数必须是 chan（通道）、map（字典）、slice（切片）中的一个，并且返回值也是类型本身。

> 注意：make 函数只用于 map，slice 和 channel，并且不返回指针。如果想要获得一个显式的指针，可以使用 new 函数进行分配，或者显式地使用一个变量的地址。

Go语言中的 new 和 make 主要区别如下：

- make 只能用来分配及初始化类型为 slice、map、chan 的数据。new 可以分配任意类型的数据；
- new 分配返回的是指针，即类型 *Type。make 返回引用，即 Type；
- new 分配的空间被清零。make 分配空间后，会进行初始化；

### 实现原理

接下来我们将分别介绍一下 make 和 new 在初始化不同[数据结构](http://c.biancheng.net/data_structure/)时的具体过程，我们会从编译期间和运行时两个不同的阶段理解这两个关键字的原理。

#### make

我们已经了解了 make 在创建 slice、map 和 channel 的具体过程，所以在这里我们也只是会简单提及 make 相关的数据结构初始化原理。



![img](http://c.biancheng.net/uploads/allimg/190903/4-1ZZ31I35HJ.gif)


在编译期的类型检查阶段，Go语言其实就将代表 make 关键字的 OMAKE 节点根据参数类型的不同转换成了 OMAKESLICE、OMAKEMAP 和 OMAKECHAN 三种不同类型的节点，这些节点最终也会调用不同的运行时函数来初始化数据结构。

#### new

内置函数 new 会在编译期的 SSA 代码生成阶段经过 callnew 函数的处理，如果请求创建的类型大小是 0，那么就会返回一个表示空指针的 zerobase 变量，在遇到其他情况时会将关键字转换成 newobject：

```go
func callnew(t *types.Type) *Node {
    if t.NotInHeap() {
        yyerror("%v is go:notinheap; heap allocation disallowed", t)
    }
    dowidth(t)
    if t.Size() == 0 {
        z := newname(Runtimepkg.Lookup("zerobase"))
        z.SetClass(PEXTERN)
        z.Type = t
        return typecheck(nod(OADDR, z, nil), ctxExpr)
    }
    fn := syslook("newobject")
    fn = substArgTypes(fn, t)
    v := mkcall1(fn, types.NewPtr(t), nil, typename(t))
    v.SetNonNil(true)
    return 
}
```

需要提到的是，哪怕当前变量是使用 var 进行初始化，在这一阶段也可能会被转换成 newobject 的函数调用并在堆上申请内存：

```go
func walkstmt(n *Node) *Node {
    switch n.Op {
    case ODCL:
        v := n.Left
        if v.Class() == PAUTOHEAP {
            if prealloc[v] == nil {
                prealloc[v] = callnew(v.Type)
            }
            nn := nod(OAS, v.Name.Param.Heapaddr, prealloc[v])
            nn.SetColas(true)
            nn = typecheck(nn, ctxStmt)
            return walkstmt(nn)
        }
    case ONEW:
        if n.Esc == EscNone {
            r := temp(n.Type.Elem())
            r = nod(OAS, r, nil)
            r = typecheck(r, ctxStmt)
            init.Append(r)
            r = nod(OADDR, r.Left, nil)
            r = typecheck(r, ctxExpr)
            n = r
        } else {
            n = callnew(n.Type.Elem())
        }
    }
}
```

当然这也不是绝对的，如果当前声明的变量或者参数不需要在当前作用域外生存，那么其实就不会被初始化在堆上，而是会初始化在当前函数的栈中并随着函数调用的结束而被销毁。

newobject 函数的工作就是获取传入类型的大小并调用 mallocgc 在堆上申请一片大小合适的内存空间并返回指向这片内存空间的指针：

```go
func newobject(typ *_type) unsafe.Pointer {
    return mallocgc(typ.size, typ, true)
}
```

### 总结

最后，简单总结一下Go语言中 make 和 new 关键字的实现原理，make 关键字的主要作用是创建 slice、map 和 Channel 等内置的数据结构，而 new 的主要作用是为类型申请一片内存空间，并返回指向这片内存的指针。

## 踩坑

1.如果调用了模块外的方法使用内部自定的type，需要把变量首字母大写，否则会认定为内部变量，无法使外部得到

![image-20220111232646070](/images/image-20220111232646070.png)

![image-20220111232707249](/images/image-20220111232707249.png)

![image-20220111232730650](/images/image-20220111232730650.png)

![image-20220111232741241](/images/image-20220111232741241.png)
