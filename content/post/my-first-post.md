---
title: "go 语言学习"
date: 2018-12-30T17:12:17+08:00
---
# 安装go

## Mac OS X 安装go



### 下载安装

1.	下载安装包跟随安装  [go下载]: https://golang.org/dl/
2.	说明： 安装完成后 此安装包会将go安装到/usr/local/go/bin 目录 放到你的PATH环境变量中
3.	查看环境变量是否生效

```shell
smallwormer# awesomeProject/small » go env
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/branyang/Library/Caches/go-build"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/branyang/go"
GOPROXY=""
GORACE=""
GOROOT="/usr/local/go"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/n9/78fzg6x95z7bc_yf1x63fq140000gn/T/go-build017305881=/tmp/go-build -gno-record-gcc-switches -fno-common"
```



  Note:	在新的go版本中 GOPATH GOBIN  GOROOT等是无需手工重新设置的，安装后会自动设置

默认go二进制发行版会将go安装到 /usr/local/go 下面。也可以将go安装到不同的位置，但是这时就要使用GOROOT环境变量来指定它的安装位置。
```sh
export GOTOOT=$HOME/go
export PATH=$PATH:$GOROOT/bin
```

##  测试GO的安装

1. 创建一个 helloworld.go 文件

   ```go
   package main
   
   import "fmt"
   
   func main() {
       fmt.Printf("hello, world\n")
   }
   ```

2.	运行go 工具确认

```
smallwormer# ~ » go run hello.go
hello, world
```
如果看到了上面的信息 说明go已经被正确安装



## 卸载GO



如果需要卸载go的安装，需要删除go目录，在Linux MAC 等系统下通常为 /usr/local/go 



## 参考链接

[go官方下载链接](https://golang.org/dl/)

[go语言学习](http://docscn.studygolang.com/doc/)



