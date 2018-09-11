title: Hello, Go!
date: 2017/05/06
comments: true
tags: 
- Golang
categories: 
- Golang
---
![](https://niyaoyao.github.io/images/golang-02.png)
# Hello, Go!
最近开始放飞自我，想学点新技术，恰好接触到 Golang 语法风格和 C 的语法风格相似，就想尝试一下。于是，就有了这篇比较水的入门文章。自然，列位看官，若是觉得此文太水，求轻喷～拜谢 🙈😁。

闲言少叙，开始正文。

# 下载与安装

在 macOS（OS X） 上安装 Go 很简单，直接到 https://golang.org/dl/ 下载对应的 pkg 安装包即可。

如果是只有 terminal 的服务端，那就借助命令行来安装。

__本文主要以 CentOS 7.2 作为操作环境，macOS 可能有些许不同。__

## 下载

到具有写权限的目录下下载最新版本的安装包，并对安装包进行校验。

```sh
cd /tmp
curl -LO https://storage.googleapis.com/golang/go1.8.1.linux-amd64.tar.gz
shasum -a 256 go1.8*.tar.gz
```

## 安装

```sh
sudo tar -C /usr/local -xvzf go1.8.1.linux-amd64.tar.gz
```

## 验证
查看 Go 的版本号，即可验证 Golang 是否安装成功。

```sh
go version
go version go1.8.1 linux/amd64
```

## 设置 Go 的环境变量

### 设置环境变量

设置 Go 语言环境变量，便于执行 Go 的命令。

```sh
sudo vi /etc/profile.d/path.sh
```

追加输入以下内容。**如果，你的 Go 所安装的路径不是 /usr/local 那就替换为你实际的安装路径。**

```sh
export PATH=$PATH:/usr/local/go/bin
```

vi(vim) 打开 **~/.bash_profile**

```sh
vi ~/.bash_profile
```

编辑如下内容。
```
export GOBIN="$HOME/projects/bin"
export GOPATH="$HOME/projects/src"
```

利用 **source** 命令将最新的配置，应用到当前的 BASH 会话中。

```sh
source /etc/profile && source ~/.bash_profile
```

# Hello, World
是的，没错～万年不变的 **Hello World**。


## 配置项目目录

创建 Go 项目目录，一般 Go 目录包含 bin、pkg、src。其中，bin 用来存编译后的二进制文件，pkg 用来存包文件，src 用来存程序源码。

```
cd ~
mkdir -p ~/projects/{bin,pkg,src}
```

## 创建 hello.go

```
vi ~/projects/src/hello.go
```

打开 hello.go 文件后，输入以下内容。

```go
package main

import "fmt"

func main() {
    fmt.Printf("Hello, World!\n")
}
```

编辑完文件后，在终端输入如下命令。

```
go install $GOPATH/hello.go
$GOBIN/hello
```

查看 Go 的手册可以得知 **go install** 是用来编译以及安装依赖包的。

```
Usage:
	go command [arguments]
	install     compile and install packages and dependencies
```
之后，在终端就会输出 **Hello, World!** 了。

## 搭建 Go 的 Web 服务
写完 **Hello, World!** 之后接下来该做什么呢？既然，我们能让 **Hello, World!** 在终端输出，是否能够让它在浏览器里输出呢？当然必须可以的～那么，接下来就开始搭建 Go 的服务端。

### 监听 TCP 端口
与 PHP 不同， Go 不需要 Nginx、Apache 这样的服务器。Go就是不需要这些，因为他直接就监听 TCP 端口了，做了 Nginx 做的事情。

#### 修改
所以，接下来就写代码，把 hello.go 稍加修改。

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "html/template"
)

func helloNY (w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello Go, NY")
}

func main() {
    http.HandleFunc("/", helloNY)

    err := http.ListenAndServe(":9090", nil)
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}
```
```
以上这段代码，新引入了 log net/http html/template 这几个依赖包。

http.HandleFunc("/", helloNY) 这句代码用来设置访问的路由，后面我们可以利用它来自定义 router。

err := http.ListenAndServe(":9090", nil) 这句代码用来设置监听的端口。

fmt.Fprintf(w, "Hello Go, NY") 这句代码用来向客户端（浏览器）输出 Hello Go, NY
```

重新编译 hello.go 终端输出结果，那我们的文件修改就完成了。但是，问题又来了。如何运行 Go 的进程，并且不中断终端的其他操作呢？

这里就需要安装一个新的进程守护软件「supervisor」。

### Supervisor 管理进程

**Supervisor is a client/server system that allows its users to control a number of processes on UNIX-like operating systems.**

Supervisor 是一款类 UNIX 操作系统上，用来管理多个进程的客户端／服务端的系统。
项目的开源仓库在 https://github.com/Supervisor/supervisor ，使用配置也比较简单，接下来就开始学习 Supervisor 来管理我们 hello.go 的进程。

#### 安装
在 CentOS 上利用 **yum** 安装 python 的工具，用 **easy_install** 安装 **supervisor**。

```sh
sudo yum install python-setuptools
sudo easy_install supervisor
```
#### 配置 supervisord.conf

输入以下命令，生成配置文件。

```
sudo echo_supervisord_conf > /etc/supervisord.conf
```

修改配置文件，配置 Go 应用。

```
[program:golang-http-server]
command=/root/projects/bin/hello
autostart=true
autorestart=true
startsecs=10
stdout_logfile=/var/log/simple_http_server.log
stdout_logfile_maxbytes=1MB
stdout_logfile_backups=10
stdout_capture_maxbytes=1MB
stderr_logfile=/var/log/simple_http_server.log
stderr_logfile_maxbytes=1MB
stderr_logfile_backups=10
stderr_capture_maxbytes=1MB
```

#### Supervisor 的客户端和服务端

开始使用 Supervisor 之前要先分清 Supervisor 的客户端和服务端。

笔者就是一开始没有分清这两个工具，一开始就不懂怎么在修改完 *.go 程序源码之后重启进程让服务更新。毕竟，Go 不像 PHP 一样可以重启 Nginx、Apache 服务器，而是通过 Supervisor 来管理进程，所以必须要搞清楚谁是 Client ，谁是 Server。

- 客户端

supervisorctl 是 Supervisor 的客户端，一般我们用 **supervisorctl** 来管理配置好的应用进程。
终端输入 **supervisorctl --help** 来查看 **supervisorctl** 用法。

```sh
supervisorctl --help
supervisorctl -- control applications run by supervisord from the cmd line.
....
```
因此，可以确定 **supervisorctl** 是管理应用的客户端。

- 服务端

同样，来看服务端。
```sh
supervisord --help
supervisord -- run a set of applications as daemons.
....
```
由此，更加确定 **supervisord** 是将应用作为后台程序运行的服务端。

#### 管理 Go 进程
开启、重启、停止 Go 进程都在 **supervisorctl** 中进行。在终端直接输入 **supervisorctl** 则可进入 Supervisor 的客户端。

```
supervisorctl
golang-http-server               RUNNING   pid 3321, uptime 5 days, 5:55:24
```
可以看到，我们刚才配置的 **golang-http-server** 正在运行。

继续输入 **help** 可以看到默认的所有命令。

```
supervisor> help

default commands (type help <topic>):
=====================================
add    clear  fg        open  quit    remove  restart   start   stop  update 
avail  exit   maintail  pid   reload  reread  shutdown  status  tail  version

supervisor> 
```

比如，当我们重新修改 hello.go 文件的话，就直接可以使用 **supervisorctl** 来重启 **golang-http-server** 这个应用的进程。

除此之外，当我们对 supervisord.conf 文件进行修改。
```
vim /etc/supervisord.conf
```

要从新配置相关文件，则须如下命令。
```
/usr/bin/supervisord -c /etc/supervisord.conf
```
两条命令的顺序不要错，不然会报 Python 和 Unix 错误。

那么，到此为止，我们访问服务器的 IP／域名:9090 端口就可以看到 **Hello Go, NY** 这个输出了。然而，问题又来了，我们可以像客户端（浏览器）输出一段 raw string ，那我们是否可以直接解析 HTML 文档，并向浏览器输出呢？ 当然仍是可以的，接下来就继续看如何加载 HTML。

#### 输出 HTML
##### 编辑 HTML 文档

输入以下 HTML 并保存为 ny-home.html 文档。
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>NY</title>
<style>
  html, body {
      color: #373D49;
      font-size: 1rem;
      line-height: 2rem;
      font-family: Georgia,Cambria,serif;
      margin: 0;
    }
    h1, h2, h3, h4, h5, h6 {
      font-family: "Source Sans Pro","Helvetica Neue",Helvetica,Arial,sans-serif;
    }
    #preview a {
      color: #A0AABF;
      text-decoration: none;
    }
    img#qrcode {
      position: absolute;
      top: 0;
      right: 0;
    }
    img#banner {
      width: 100%;
    }
    .wrapper {
      margin: 10px;
    }
</style>
</head>
<body id="preview">
<div>
<img id="banner" src="http://chuantu.biz/t5/80/1494091793x1729546437.jpg" alt="">
</div>
<div class="wrapper">
  <p>Welcome to NY space.<br>
  Here are some keywords about me.<br>
  Sensitive | Thoughtful | Idealism | ❤️ @SuneBear | Cear’s mommy~ 🐱</p>
  <p>Wanna know more about me? You can go below<br>
  <a href="https://niyaoyao.github.io/">https://niyaoyao.github.io/</a>
  </p>
</div>
</body>
</html>
```

##### 修改 hello.go

```go
package main

import (
    "fmt"
    "log"
    "net/http"
    "html/template"
)

func sayHello (w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello Go, NY")
}

func nyHome(w http.ResponseWriter, r *http.Request)  {
  if r.Method == "GET" {
      t, _ := template.ParseFiles("ny-home.html")
      log.Println(t.Execute(w, nil))
  } 
}

func main() {

    http.HandleFunc("/", nyHome)
    http.HandleFunc("/hello", sayHello)

    err := http.ListenAndServe(":9090", nil)
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}
```

修改并保存 hello.go 文件后，重新编译出二进制文件。
```
go install $GOPATH/hello.go
```

编译成功后，用 **supervisorctl** 重启应用进程。
```
supervisorctl 
golang-http-server               RUNNING   pid 3321, uptime 5 days, 7:10:30
supervisor> restart golang-http-server 
golang-http-server: stopped
golang-http-server: started
supervisor> 
```

最后访问 http://www.niyao.space:9090/ 就可以看到最终的成果啦。
到此，如何搭建一个 Golang 的 Web 服务就算比较完满的入门了。当然，我的习惯是现做应用小 demo 然后再慢慢熟悉语言的具体语法细节，所以，还是 Go 小白一枚，如果文中有错误，欢迎指出十分感谢，可以关注我的公众号向我反馈，十分感谢 😁 ～
![](https://niyaoyao.github.io/images/qrcod_ny_1.jpg)

# 参考资料
- **How To Install Go 1.7 on CentOS 7** https://www.digitalocean.com/community/tutorials/how-to-install-go-1-7-on-centos-7
- **supervisor 运行 golang 守护进程** http://www.01happy.com/supervisor-golang-daemon/
- **Go Web 编程** https://astaxie.gitbooks.io/build-web-application-with-golang/zh/