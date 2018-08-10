---
layout: post
title: 'Setting Up a Go Development Environment'
description: "如何配置Go开发环境"
date: 2018-03-21
author: MartyPang
cover: '/assets/img/go/golang.png'
tags: Go
---

> 如何配置Go语言开发环境

### 1 安装 Go

#### Go的安装方式

Go有多种安装方式，你可以选择自己喜欢的。这里我们介绍三种最常见的安装方式：

- Go源码安装：这是一种标准的软件安装方式。对于经常使用Unix类系统的用户，尤其对于开发者来说，从源码安装可以自己定制。
- Go标准包安装：Go提供了方便的安装包，支持Windows、Linux、Mac等系统。这种方式适合快速安装，可根据自己的系统位数下载好相应的安装包，一路next就可以轻松安装了。**推荐这种方式**

##### Go源码安装

Go 1.5彻底移除C代码，Runtime、Compiler、Linker均由Go编写,实现自举。只需要安装了上一个版本,即可从源码安装。

在Go 1.5前,Go的源代码中，有些部分是用Plan 9 C和AT&T汇编写的，因此假如你要想从源码安装，就必须安装C的编译工具。

在Mac系统中，只要你安装了Xcode，就已经包含了相应的编译工具。

在类Unix系统中，需要安装gcc等工具。例如Ubuntu系统可通过在终端中执行`sudo apt-get install gcc libc6-dev`来安装编译工具。

在Windows系统中，你需要安装MinGW，然后通过MinGW安装gcc，并设置相应的环境变量。

你可以直接去官网[下载源码](http://golang.org/dl/)，找相应的`goVERSION.src.tar.gz`的文件下载，下载之后解压缩到`$HOME`目录，执行如下代码：

```
cd go/src
./all.bash
```

运行all.bash后出现"ALL TESTS PASSED"字样时才算安装成功。

上面是Unix风格的命令，Windows下的安装方式类似，只不过是运行`all.bat`，调用的编译器是MinGW的gcc。

如果是Mac或者Unix用户需要设置几个环境变量，如果想重启之后也能生效的话把下面的命令写到`.bashrc`或者`.zshrc`里面，

```
export GOPATH=$HOME/gopath
export PATH=$PATH:$HOME/go/bin:$GOPATH/bin
```

如果你是写入文件的，记得执行`bash .bashrc`或者`bash .zshrc`使得设置立马生效。

如果是window系统，就需要设置环境变量，在path里面增加相应的go所在的目录，设置gopath变量。

当你设置完毕之后在命令行里面输入`go`，看到如下图片即说明你已经安装成功

![](/assets/img/go/1.1.mac.png)

图1.1 源码安装之后执行Go命令的图

如果出现Go的Usage信息，那么说明Go已经安装成功了；如果出现该命令不存在，那么可以检查一下自己的PATH环境变中是否包含了Go的安装目录。

> 关于上面的GOPATH将在下面小节详细讲解

##### Go标准包安装（推荐）

Go提供了每个平台打好包的一键安装，这些包默认会安装到如下目录：/usr/local/go (Windows系统：c:\Go)，当然你可以改变他们的安装位置，但是改变之后你必须在你的环境变量中设置如下信息：

```
export GOROOT=$HOME/go  
export GOPATH=$HOME/gopath
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

上面这些命令对于Mac和Unix用户来说最好是写入`.bashrc`或者`.zshrc`文件，对于windows用户来说当然是写入环境变量。

###### 如何判断自己的操作系统是32位还是64位？

我们接下来的Go安装需要判断操作系统的位数，所以这小节我们先确定自己的系统类型。

Windows系统用户请按Win+R运行cmd，输入`systeminfo`后回车，稍等片刻，会出现一些系统信息。在“系统类型”一行中，若显示“x64-based PC”，即为64位系统；若显示“X86-based PC”，则为32位系统。

Mac系统用户建议直接使用64位的，因为Go所支持的Mac OS X版本已经不支持纯32位处理器了。

Linux系统用户可通过在Terminal中执行命令`arch`(即`uname -m`)来查看系统信息：

64位系统显示

```
x86_64
```

32位系统显示

```
i386
```

###### Mac 安装

访问[下载地址][downlink]，32位系统下载go1.4.2.darwin-386-osx10.8.pkg(最新版已无32位下载)，64位系统下载go1.7.4.darwin-amd64.pkg，双击下载文件，一路默认安装点击下一步，这个时候go已经安装到你的系统中，默认已经在PATH中增加了相应的`~/go/bin`,这个时候打开终端，输入`go`

看到类似上面源码安装成功的图片说明已经安装成功

如果出现go的Usage信息，那么说明go已经安装成功了；如果出现该命令不存在，那么可以检查一下自己的PATH环境变中是否包含了go的安装目录。

###### Linux 安装

访问[下载地址][downlink]，32位系统下载go1.7.4.linux-386.tar.gz，64位系统下载go1.7.4.linux-amd64.tar.gz，

假定你想要安装Go的目录为 `$GO_INSTALL_DIR`，后面替换为相应的目录路径。

解压缩`tar.gz`包到安装目录下：`tar zxvf go1.7.4.linux-amd64.tar.gz -C $GO_INSTALL_DIR`。

设置PATH，`export PATH=$PATH:$GO_INSTALL_DIR/go/bin`

然后执行`go`

![](/assets/img/go/1.1.linux.png)

图1.2 Linux系统下安装成功之后执行go显示的信息

如果出现go的Usage信息，那么说明go已经安装成功了；如果出现该命令不存在，那么可以检查一下自己的PATH环境变中是否包含了go的安装目录。

###### Windows 安装

访问[Google Code 下载页][downlink]，32 位请选择名称中包含 windows-386 的 msi 安装包，64 位请选择名称中包含 windows-amd64 的。下载好后运行，不要修改默认安装目录 C:\Go\，若安装到其他位置会导致不能执行自己所编写的 Go 代码。安装完成后默认会在环境变量 Path 后添加 Go 安装目录下的 bin 目录 `C:\Go\bin\`，并添加环境变量 GOROOT，值为 Go 安装根目录 `C:\Go\` 。

**验证是否安装成功**

在运行中输入 `cmd` 打开命令行工具，在提示符下输入 `go`，检查是否能看到 Usage 信息。输入 `cd %GOROOT%`，看是否能进入 Go 安装目录。若都成功，说明安装成功。

不能的话请检查上述环境变量 Path 和 GOROOT 的值。若不存在请卸载后重新安装，存在请重启计算机后重试以上步骤。

[downlink]: http://golang.org/dl/	"Go安装包下载"

### 2 GOPATH与工作空间

前面我们在安装Go的时候看到需要设置GOPATH变量，Go从1.1版本开始必须设置这个变量，而且不能和Go的安装目录一样，这个目录用来存放Go源码，Go的可运行文件，以及相应的编译之后的包文件。所以这个目录下面有三个子目录：src、bin、pkg

#### GOPATH设置

  go 命令依赖一个重要的环境变量：$GOPATH

  Windows系统中环境变量的形式为`%GOPATH%`，本书主要使用Unix形式，Windows用户请自行替换。

  *（注：这个不是Go安装目录。下面以笔者的工作目录为示例，如果你想不一样请把GOPATH替换成你的工作目录。）*

  在类似 Unix 环境大概这样设置：

```sh
export GOPATH=/home/apple/mygo
```

  为了方便，应该新建以上文件夹，并且上一行加入到 `.bashrc` 或者 `.zshrc` 或者自己的 `sh` 的配置文件中。

  Windows 设置如下，新建一个环境变量名称叫做GOPATH：

```sh
	GOPATH=c:\mygo
```

GOPATH允许多个目录，当有多个目录时，请注意分隔符，多个目录的时候Windows是分号，Linux系统是冒号，当有多个GOPATH时，默认会将go get的内容放在第一个目录下。

以上 $GOPATH 目录约定有三个子目录：

- src 存放源代码（比如：.go .c .h .s等）
- pkg 编译后生成的文件（比如：.a）
- bin 编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中，如果有多个gopath，那么使用`${GOPATH//://bin:}/bin`添加所有的bin目录）

以后我所有的例子都是以mygo作为我的gopath目录

#### 代码目录结构规划

GOPATH下的src目录就是接下来开发程序的主要目录，所有的源码都是放在这个目录下面，那么一般我们的做法就是一个目录一个项目，例如: $GOPATH/src/mymath 表示mymath这个应用包或者可执行应用，这个根据package是main还是其他来决定，main的话就是可执行应用，其他的话就是应用包，这个会在后续详细介绍package。

所以当新建应用或者一个代码包时都是在src目录下新建一个文件夹，文件夹名称一般是代码包名称，当然也允许多级目录，例如在src下面新建了目录$GOPATH/src/github.com/astaxie/beedb 那么这个包路径就是"github.com/astaxie/beedb"，包名称是最后一个目录beedb

下面我就以mymath为例来讲述如何编写应用包，执行如下代码

```sh
cd $GOPATH/src
mkdir mymath
```

新建文件sqrt.go，内容如下

```go
// $GOPATH/src/mymath/sqrt.go源码如下：
package mymath

func Sqrt(x float64) float64 {
	z := 0.0
	for i := 0; i < 1000; i++ {
		z -= (z*z - x) / (2 * x)
	}
	return z
}
```

这样我的应用包目录和代码已经新建完毕，注意：一般建议package的名称和目录名保持一致

#### 编译应用

上面我们已经建立了自己的应用包，如何进行编译安装呢？有两种方式可以进行安装

1、只要进入对应的应用包目录，然后执行`go install`，就可以安装了

2、在任意的目录执行如下代码`go install mymath`

安装完之后，我们可以进入如下目录

```sh
cd $GOPATH/pkg/${GOOS}_${GOARCH}
//可以看到如下文件
mymath.a
```

这个.a文件是应用包，那么我们如何进行调用呢？

接下来我们新建一个应用程序来调用这个应用包

新建应用包mathapp

```sh
cd $GOPATH/src
mkdir mathapp
cd mathapp
vim main.go
```

`$GOPATH/src/mathapp/main.go`源码：

```go
package main

import (
	  "mymath"
	  "fmt"
)

func main() {
	  fmt.Printf("Hello, world.  Sqrt(2) = %v\n", mymath.Sqrt(2))
}
```

可以看到这个的package是`main`，import里面调用的包是`mymath`,这个就是相对于`$GOPATH/src`的路径，如果是多级目录，就在import里面引入多级目录，如果你有多个GOPATH，也是一样，Go会自动在多个`$GOPATH/src`中寻找。

如何编译程序呢？进入该应用目录，然后执行`go build`，那么在该目录下面会生成一个mathapp的可执行文件

```sh
./mathapp
```

输出如下内容

```sh
Hello, world.  Sqrt(2) = 1.414213562373095
```

如何安装该应用，进入该目录执行`go install`,那么在$GOPATH/bin/下增加了一个可执行文件mathapp, 还记得前面我们把`$GOPATH/bin`加到我们的PATH里面了，这样可以在命令行输入如下命令就可以执行

```sh
mathapp
```

也是输出如下内容

```
Hello, world.  Sqrt(2) = 1.414213562373095
```

这里我们展示如何编译和安装一个可运行的应用，以及如何设计我们的目录结构。

#### 获取远程包

   go语言有一个获取远程包的工具就是`go get`，目前go get支持多数开源社区(例如：github、googlecode、bitbucket、Launchpad)

```
go get github.com/astaxie/beedb
```

> go get -u 参数可以自动更新包，而且当go get的时候会自动获取该包依赖的其他第三方包

通过这个命令可以获取相应的源码，对应的开源平台采用不同的源码控制工具，例如github采用git、googlecode采用hg，所以要想获取这些源码，必须先安装相应的源码控制工具

通过上面获取的代码在我们本地的源码相应的代码结构如下

```
$GOPATH
  src
   |--github.com
		  |-astaxie
			  |-beedb
   pkg
	|--相应平台
		 |-github.com
			   |--astaxie
					|beedb.a
```

go get本质上可以理解为首先第一步是通过源码工具clone代码到src下面，然后执行`go install`

在代码中如何使用远程包，很简单的就是和使用本地包一样，只要在开头import相应的路径就可以

```
import "github.com/astaxie/beedb"
```

#### 程序的整体结构

通过上面建立的我本地的mygo的目录结构如下所示

```
bin/
	mathapp
pkg/
	平台名/ 如：darwin_amd64、linux_amd64
		 mymath.a
		 github.com/
			  astaxie/
				   beedb.a
src/
	mathapp
		  main.go
	mymath/
		  sqrt.go
	github.com/
		   astaxie/
				beedb/
					beedb.go
					util.go
```

从上面的结构我们可以很清晰的看到，bin目录下面存的是编译之后可执行的文件，pkg下面存放的是应用包，src下面保存的是应用源代码

### 3 Go 命令

Go语言自带有一套完整的命令操作工具，你可以通过在命令行中执行`go`来查看它们：

  ![](/assets/img/go/1.1.mac.png)

图1.3 Go命令显示详细的信息

  这些命令对于我们平时编写的代码非常有用，接下来就让我们了解一些常用的命令。

#### go build

  这个命令主要用于编译代码。在包的编译过程中，若有必要，会同时编译与之相关联的包。

- 如果是普通包，就像我们在1.2节中编写的`mymath`包那样，当你执行`go build`之后，它不会产生任何文件。如果你需要在`$GOPATH/pkg`下生成相应的文件，那就得执行`go install`。

- 如果是`main`包，当你执行`go build`之后，它就会在当前目录下生成一个可执行文件。如果你需要在`$GOPATH/bin`下生成相应的文件，需要执行`go install`，或者使用`go build -o 路径/a.exe`。

  - 如果某个项目文件夹下有多个文件，而你只想编译某个文件，就可在`go build`之后加上文件名，例如`go build a.go`；`go build`命令默认会编译当前目录下的所有go文件。
  - 你也可以指定编译输出的文件名。例如1.2节中的`mathapp`应用，我们可以指定`go build -o astaxie.exe`，默认情况是你的package名(非main包)，或者是第一个源文件的文件名(main包)。

  （注：实际上，package名在[Go语言规范](https://golang.org/ref/spec)中指代码中“package”后使用的名称，此名称可以与文件夹名不同。默认生成的可执行文件名是文件夹名。）

  - go build会忽略目录下以“_”或“.”开头的go文件。
  - 如果你的源代码针对不同的操作系统需要不同的处理，那么你可以根据不同的操作系统后缀来命名文件。例如有一个读取数组的程序，它对于不同的操作系统可能有如下几个源文件：

  array_linux.go
  array_darwin.go
  array_windows.go
  array_freebsd.go

  `go build`的时候会选择性地编译以系统名结尾的文件（Linux、Darwin、Windows、Freebsd）。例如Linux系统下面编译只会选择array_linux.go文件，其它系统命名后缀文件全部忽略。

参数的介绍

- `-o` 指定输出的文件名，可以带上路径，例如 `go build -o a/b/c`
- `-i` 安装相应的包，编译+`go install`
- `-a` 更新全部已经是最新的包的，但是对标准包不适用
- `-n` 把需要执行的编译命令打印出来，但是不执行，这样就可以很容易的知道底层是如何运行的
- `-p n` 指定可以并行可运行的编译数目，默认是CPU数目
- `-race` 开启编译的时候自动检测数据竞争的情况，目前只支持64位的机器
- `-v` 打印出来我们正在编译的包名
- `-work` 打印出来编译时候的临时文件夹名称，并且如果已经存在的话就不要删除
- `-x` 打印出来执行的命令，其实就是和`-n`的结果类似，只是这个会执行
- `-ccflags 'arg list'` 传递参数给5c, 6c, 8c 调用
- `-compiler name` 指定相应的编译器，gccgo还是gc
- `-gccgoflags 'arg list'` 传递参数给gccgo编译连接调用
- `-gcflags 'arg list'` 传递参数给5g, 6g, 8g 调用
- `-installsuffix suffix` 为了和默认的安装包区别开来，采用这个前缀来重新安装那些依赖的包，`-race`的时候默认已经是`-installsuffix race`,大家可以通过`-n`命令来验证
- `-ldflags 'flag list'` 传递参数给5l, 6l, 8l 调用
- `-tags 'tag list'` 设置在编译的时候可以适配的那些tag，详细的tag限制参考里面的 [Build Constraints](http://golang.org/pkg/go/build/)

#### go clean

  这个命令是用来移除当前源码包和关联源码包里面编译生成的文件。这些文件包括

```
_obj/            旧的object目录，由Makefiles遗留
_test/           旧的test目录，由Makefiles遗留
_testmain.go     旧的gotest文件，由Makefiles遗留
test.out         旧的test记录，由Makefiles遗留
build.out        旧的test记录，由Makefiles遗留
*.[568ao]        object文件，由Makefiles遗留

DIR(.exe)        由go build产生
DIR.test(.exe)   由go test -c产生
MAINFILE(.exe)   由go build MAINFILE.go产生
*.so             由 SWIG 产生
```

  我一般都是利用这个命令清除编译文件，然后github递交源码，在本机测试的时候这些编译文件都是和系统相关的，但是对于源码管理来说没必要。

```
$ go clean -i -n
cd /Users/astaxie/develop/gopath/src/mathapp
rm -f mathapp mathapp.exe mathapp.test mathapp.test.exe app app.exe
rm -f /Users/astaxie/develop/gopath/bin/mathapp
```

参数介绍

- `-i` 清除关联的安装的包和可运行文件，也就是通过go install安装的文件
- `-n` 把需要执行的清除命令打印出来，但是不执行，这样就可以很容易的知道底层是如何运行的
- `-r` 循环的清除在import中引入的包
- `-x` 打印出来执行的详细命令，其实就是`-n`打印的执行版本

#### go fmt

  有过C/C++经验的读者会知道,一些人经常为代码采取K&R风格还是ANSI风格而争论不休。在go中，代码则有标准的风格。由于之前已经有的一些习惯或其它的原因我们常将代码写成ANSI风格或者其它更合适自己的格式，这将为人们在阅读别人的代码时添加不必要的负担，所以go强制了代码格式（比如左大括号必须放在行尾），不按照此格式的代码将不能编译通过，为了减少浪费在排版上的时间，go工具集中提供了一个`go fmt`命令 它可以帮你格式化你写好的代码文件，使你写代码的时候不需要关心格式，你只需要在写完之后执行`go fmt <文件名>.go`，你的代码就被修改成了标准格式，但是我平常很少用到这个命令，因为开发工具里面一般都带了保存时候自动格式化功能，这个功能其实在底层就是调用了`go fmt`。接下来的一节我将讲述两个工具，这两个工具都自带了保存文件时自动化`go fmt`功能。

使用go fmt命令，其实是调用了gofmt，而且需要参数-w，否则格式化结果不会写入文件。gofmt -w -l src，可以格式化整个项目。

所以go fmt是gofmt的上层一个包装的命令，我们想要更多的个性化的格式化可以参考 [gofmt](http://golang.org/cmd/gofmt/)

gofmt的参数介绍

- `-l` 显示那些需要格式化的文件
- `-w` 把改写后的内容直接写入到文件中，而不是作为结果打印到标准输出。
- `-r` 添加形如“a[b:len(a)] -> a[b:]”的重写规则，方便我们做批量替换
- `-s` 简化文件中的代码
- `-d` 显示格式化前后的diff而不是写入文件，默认是false
- `-e` 打印所有的语法错误到标准输出。如果不使用此标记，则只会打印不同行的前10个错误。
- `-cpuprofile` 支持调试模式，写入相应的cpufile到指定的文件

#### go get

  这个命令是用来动态获取远程代码包的，目前支持的有BitBucket、GitHub、Google Code和Launchpad。这个命令在内部实际上分成了两步操作：第一步是下载源码包，第二步是执行`go install`。下载源码包的go工具会自动根据不同的域名调用不同的源码工具，对应关系如下：

```
BitBucket (Mercurial Git)
GitHub (Git)
Google Code Project Hosting (Git, Mercurial, Subversion)
Launchpad (Bazaar)
```

  所以为了`go get` 能正常工作，你必须确保安装了合适的源码管理工具，并同时把这些命令加入你的PATH中。其实`go get`支持自定义域名的功能，具体参见`go help remote`。

参数介绍：

- `-d` 只下载不安装
- `-f` 只有在你包含了`-u`参数的时候才有效，不让`-u`去验证import中的每一个都已经获取了，这对于本地fork的包特别有用
- `-fix` 在获取源码之后先运行fix，然后再去做其他的事情
- `-t` 同时也下载需要为运行测试所需要的包
- `-u` 强制使用网络去更新包和它的依赖包
- `-v` 显示执行的命令

#### go install

  这个命令在内部实际上分成了两步操作：第一步是生成结果文件(可执行文件或者.a包)，第二步会把编译好的结果移到`$GOPATH/pkg`或者`$GOPATH/bin`。

参数支持`go build`的编译参数。大家只要记住一个参数`-v`就好了，这个随时随地的可以查看底层的执行信息。

#### go test

  执行这个命令，会自动读取源码目录下面名为`*_test.go`的文件，生成并运行测试用的可执行文件。输出的信息类似

```
ok   archive/tar   0.011s
FAIL archive/zip   0.022s
ok   compress/gzip 0.033s
...
```

  默认的情况下，不需要任何的参数，它会自动把你源码包下面所有test文件测试完毕，当然你也可以带上参数，详情请参考`go help testflag`

这里我介绍几个我们常用的参数：

- `-bench regexp` 执行相应的benchmarks，例如 `-bench=.`
- `-cover` 开启测试覆盖率
- `-run regexp` 只运行regexp匹配的函数，例如 `-run=Array` 那么就执行包含有Array开头的函数
- `-v` 显示测试的详细命令

#### go tool

`go tool`下面下载聚集了很多命令，这里我们只介绍两个，fix和vet

- `go tool fix .` 用来修复以前老版本的代码到新版本，例如go1之前老版本的代码转化到go1,例如API的变化
- `go tool vet directory|files` 用来分析当前目录的代码是否都是正确的代码,例如是不是调用fmt.Printf里面的参数不正确，例如函数里面提前return了然后出现了无用代码之类的。

#### go generate

这个命令是从Go1.4开始才设计的，用于在编译前自动化生成某类代码。`go generate`和`go build`是完全不一样的命令，通过分析源码中特殊的注释，然后执行相应的命令。这些命令都是很明确的，没有任何的依赖在里面。而且大家在用这个之前心里面一定要有一个理念，这个`go generate`是给你用的，不是给使用你这个包的人用的，是方便你来生成一些代码的。

这里我们来举一个简单的例子，例如我们经常会使用`yacc`来生成代码，那么我们常用这样的命令：

```
go tool yacc -o gopher.go -p parser gopher.y
```

-o 指定了输出的文件名， -p指定了package的名称，这是一个单独的命令，如果我们想让`go generate`来触发这个命令，那么就可以在当然目录的任意一个`xxx.go`文件里面的任意位置增加一行如下的注释：

```
//go:generate go tool yacc -o gopher.go -p parser gopher.y
```

这里我们注意了，`//go:generate`是没有任何空格的，这其实就是一个固定的格式，在扫描源码文件的时候就是根据这个来判断的。

所以我们可以通过如下的命令来生成，编译，测试。如果`gopher.y`文件有修改，那么就重新执行`go generate`重新生成文件就好。

```
$ go generate
$ go build
$ go test
```

#### godoc

在Go1.2版本之前还支持`go doc`命令，但是之后全部移到了godoc这个命令下，需要这样安装`go get golang.org/x/tools/cmd/godoc`

  很多人说go不需要任何的第三方文档，例如chm手册之类的（其实我已经做了一个了，[chm手册](https://github.com/astaxie/godoc)），因为它内部就有一个很强大的文档工具。

  如何查看相应package的文档呢？
  例如builtin包，那么执行`godoc builtin`
  如果是http包，那么执行`godoc net/http`
  查看某一个包里面的函数，那么执行`godoc fmt Printf`
  也可以查看相应的代码，执行`godoc -src fmt Printf`

  通过命令在命令行执行 godoc -http=:端口号 比如`godoc -http=:8080`。然后在浏览器中打开`127.0.0.1:8080`，你将会看到一个golang.org的本地copy版本，通过它你可以查询pkg文档等其它内容。如果你设置了GOPATH，在pkg分类下，不但会列出标准包的文档，还会列出你本地`GOPATH`中所有项目的相关文档，这对于经常被墙的用户来说是一个不错的选择。

#### 其它命令

  go还提供了其它很多的工具，例如下面的这些工具

```
go version 查看go当前的版本
go env 查看当前go的环境变量
go list 列出当前全部安装的package
go run 编译并运行Go程序
```

以上这些工具还有很多参数没有一一介绍，用户可以使用`go help 命令`获取更详细的帮助信息。



### 4 推荐使用的IDE

#### Gogland

Gogland是JetBrains公司推出的Go语言集成开发环境，是Idea Go插件是强化版。Gogland同样基于IntelliJ平台开发，支持JetBrains的插件体系。

下载地址:https://www.jetbrains.com/go/

#### Atom

Atom是Github基于Electron和web技术构建的开源编辑器, 是一款很漂亮强大的编辑器缺点是速度比较慢。

首先要先安装下Atom，下载地址: https://atom.io/

然后安装go-plus插件:

```
go-plus是Atom上面的一款开源的go语言开发环境的的插件
```

```
它需要依赖下面的go语言工具:
```

```go
		1.autocomplete-go ：gocode的代码自动提示
		2.gofmt ：使用goftm,goimports,goturns
		3.builder-go:go-install 和go-test,验证代码，给出建议
		4.gometalinet-linter:goline,vet,gotype的检查
		5.navigator-godef:godef
		6.tester-goo :go test
		7.gorename :rename
```

在Atom中的 Preference 中可以找到install菜单,输入 go-plus,然后点击安装(install)

就会开始安装 go-plus ， go-plus 插件会自动安装对应的依赖插件，如果没有安装对应的go的类库会自动运行: go get 安装。

#### Visual Studio Code

vscode是微软基于Electron和web技术构建的开源编辑器, 是一款很强大的编辑器。开源地址:https://github.com/Microsoft/vscode

1、安装Visual Studio Code 最新版

官方网站：https://code.visualstudio.com/ 
下载Visual Studio Code 最新版，安装过程略。

2、安装Go插件

点击右边的Extensions图标
搜索Go插件
在插件列表中，选择 Go，进行安装，安装之后，系统会提示重启Visual Studio Code。

建议把自动保存功能开启。开启方法为：选择菜单File，点击Auto save。

vscode代码设置可用于Go扩展。这些都可以在用户的喜好来设置或工作区设置（.vscode/settings.json）。

打开首选项-用户设置settings.json:

```go
{
    "go.buildOnSave": true,
    "go.lintOnSave": true,
    "go.vetOnSave": true,
    "go.buildFlags": [],
    "go.lintFlags": [],
    "go.vetFlags": [],
    "go.coverOnSave": false,
    "go.useCodeSnippetsOnFunctionSuggest": false,
    "go.formatOnSave": true,
    //goimports
    "go.formatTool": "goreturns",
    "go.goroot": "",//你的Goroot
    "go.gopath": "",//你的Gopath
}
```

接着安装依赖包支持(网络不稳定,请直接到Github[Golang](https://github.com/golang)下载再移动到相关目录):

```go
	go get -u -v github.com/nsf/gocode
	go get -u -v github.com/rogpeppe/godef
	go get -u -v github.com/zmb3/gogetdoc
	go get -u -v github.com/golang/lint/golint
	go get -u -v github.com/lukehoban/go-outline
	go get -u -v sourcegraph.com/sqs/goreturns
	go get -u -v golang.org/x/tools/cmd/gorename
	go get -u -v github.com/tpng/gopkgs
	go get -u -v github.com/newhook/go-symbols
	go get -u -v golang.org/x/tools/cmd/guru
	go get -u -v github.com/cweill/gotests/...
```

vscode还有一项很强大的功能就是断点调试,结合[delve](https://github.com/derekparker/delve)可以很好的进行Go代码调试

```go
	go get -v -u github.com/peterh/liner github.com/derekparker/delve/cmd/dlv
	
	brew install go-delve/delve/delve(mac可选)
```

如果有问题再来一遍:

```go	
	go get -v -u github.com/peterh/liner github.com/derekparker/delve/cmd/dlv
```

注意:修改"dlv-cert"证书, 选择"显示简介"->"信任"->"代码签名" 修改为: 始终信任

打开首选项-工作区设置,配置launch.json:

```go
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "main.go",
            "type": "go",
            "request": "launch",
            "mode": "debug",
            "remotePath": "",
            "port": 2345,
            "host": "127.0.0.1",
            "program": "${workspaceRoot}",//工作空间路径
            "env": {},
            "args": [],
            "showLog": true
        }
    ]
}
```

#### Sublime Text

  这里将介绍Sublime Text 3（以下简称Sublime）+GoSublime + gocode的组合，那么为什么选择这个组合呢？

- 自动化提示代码,如下图所示

  ![](/assets/img/go/1.4.sublime1.png)

  图1.5 sublime自动化提示界面

- 保存的时候自动格式化代码，让您编写的代码更加美观，符合Go的标准。

  - 支持项目管理

  ![](/assets/img/go/1.4.sublime2.png)

  图1.6 sublime项目管理界面

  - 支持语法高亮
  - Sublime Text 3可免费使用，只是保存次数达到一定数量之后就会提示是否购买，点击取消继续用，和正式注册版本没有任何区别。

接下来就开始讲如何安装，下载[Sublime](http://www.sublimetext.com/)

  根据自己相应的系统下载相应的版本，然后打开Sublime，对于不熟悉Sublime的同学可以先看一下这篇文章[Sublime Text 全程指南](http://blog.jobbole.com/88648/)或者[sublime text3入门教程](http://blog.csdn.net/sam976/article/details/52076271)

1. 打开之后安装 Package Control：Ctrl+` 打开命令行，执行如下代码：

   适用于 Sublime Text 3：

```go
	import  urllib.request,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();urllib.request.install_opener(urllib.request.build_opener(urllib.request.ProxyHandler()));open(os.path.join(ipp,pf),'wb').write(urllib.request.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())
```

  适用于 Sublime Text 2：

```go
	import  urllib2,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();os.makedirs(ipp)ifnotos.path.exists(ipp)elseNone;urllib2.install_opener(urllib2.build_opener(urllib2.ProxyHandler()));open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read());print('Please restart Sublime Text to finish installation')
```

   这个时候重启一下Sublime，可以发现在在菜单栏多了一个如下的栏目，说明Package Control已经安装成功了。

  ![](/assets/img/go/1.4.sublime3.png)

```
图1.7 sublime包管理
```



1. 安装完之后就可以安装Sublime的插件了。需安装GoSublime、SidebarEnhancements和Go Build，安装插件之后记得重启Sublime生效，Ctrl+Shift+p打开Package Controll 输入`pcip`（即“Package Control: Install Package”的缩写）。

   这个时候看左下角显示正在读取包数据，完成之后出现如下界面

   ![](/assets/img/go/1.4.sublime4.png)

   图1.8 sublime安装插件界面

   这个时候输入GoSublime，按确定就开始安装了。同理应用于SidebarEnhancements和Go Build。

2. 安装[gocode](https://github.com/nsf/gocode/)

```go
   go get -u github.com/nsf/gocode
```
gocode 将会安装在默认`$GOBIN`
另外建议安装gotests(生成测试代码)，先在sublime安装gotests插件,再运行:
```go
go get -u -v github.com/cweill/gotests/...
```

1. 验证是否安装成功，你可以打开Sublime，打开main.go，看看语法是不是高亮了，输入`import`是不是自动化提示了，`import "fmt"`之后，输入`fmt.`是不是自动化提示有函数了。

   如果已经出现这个提示，那说明你已经安装完成了，并且完成了自动提示。

   如果没有出现这样的提示，一般就是你的`$PATH`没有配置正确。你可以打开终端，输入gocode，是不是能够正确运行，如果不行就说明`$PATH`没有配置正确。
   (针对XP)有时候在终端能运行成功,但sublime无提示或者编译解码错误,请安装sublime text3和convert utf8插件试一试

   1. MacOS下已经设置了$GOROOT, $GOPATH, $GOBIN，还是没有自动提示怎么办。

   请在sublime中使用command + 9， 然后输入env检查$PATH, GOROOT, $GOPATH, $GOBIN等变量， 如果没有请采用下面的方法。

   首先建立下面的连接， 然后从Terminal中直接启动sublime
```sh
   ln -s /Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl /usr/local/bin/sublime
```