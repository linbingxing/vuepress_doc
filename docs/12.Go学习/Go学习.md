---
title: Go学习
date: 2021-12-11 13:51:16
permalink: /pages/f7cc32/
categories:
  - Go学习
tags:
  - 
---
#  Go学习

##  优势

- 语法简洁，相比其他语言更容易上手，开发效率更高；

- 自带垃圾回收（GC），不用再手动申请释放内存，能够有效避免 Bug，提高性能；

- 语言层面的并发支持，让你很容易开发出高性能的程序；

- 提供的标准库强大，第三方库也足够丰富，可以拿来即用，提高开发效率；

- 可通过静态编译直接生成一个可执行文件，运行时不依赖其他库，部署方便，可伸缩能力强；

- 提供跨平台支持，很容易编译出跨各个系统平台直接运行的程序。

## Go 语言环境搭建

要想搭建 Go 语言开发环境，需要先下载 Go 语言开发包。你可以从官网 https://golang.org/dl/ 和 https://golang.google.cn/dl/ 下载（第一个链接是国外的官网，第二个是国内的官网，如果第一个访问不了，可以从第二个下载）。

下载时可以根据自己的操作系统选择相应的开发包，比如 Window、MacOS 或是 Linux 等，如下图所示：

![go install](https://gitee.com/linbingxing/image/raw/master/go/go%20install.png)

### Windows MSI 下安装
MSI 安装的方式比较简单，在 Windows 系统上推荐使用这种方式。现在的操作系统基本上都是 64 位的，所以选择 64 位的 go1.16.12.windows-amd64.msi 下载即可，如果操作系统是 32 位的，选择 go1.16.12.windows-386.msi 进行下载。

下载后双击该 MSI 安装文件，按照提示一步步地安装即可。在默认情况下，Go 语言开发工具包会被安装到 c:\Go 目录，你也可以在安装过程中选择自己想要安装的目录。

假设安装到 c:\Go 目录，安装程序会自动把 c:\Go\bin 添加到你的 PATH 环境变量中，如果没有的话，你可以通过系统 -> 控制面板 -> 高级 -> 环境变量选项来手动添加。

### Linux 下安装
Linux 系统同样有 32 位和 64 位，你可以根据你的 Linux 操作系统选择相应的压缩包，它们分别是 go1.16.12.linux-386.tar.gz 和 go1.16.12.linux-amd64.tar.gz。

下载成功后，需要先进行解压，假设你下载的是 go1.16.12.linux-amd64.tar.gz，在终端通过如下命令即可解压：

```shell
sudo tar -C /usr/local -xzf go1.16.12.linux-amd64.tar.gz
```

输入后回车，然后输入你的电脑密码，即可解压到 /usr/local 目录，然后把 /usr/local/go/bin 添加到 PATH 环境变量中，就可以使用 Go 语言开发工具包了。

把下面这段添加到 /etc/profile 或者 $HOME/.profile 文件中，保存后退出即可成功添加环境变量。

```shell
export PATH=$PATH:/usr/local/go/bin
```

###  macOS 下安装
如果你的操作系统是 macOS，可以采用 PKG 安装包。下载 go1.16.12.darwin-amd64.pkg 后，双击按照提示安装即可。安装成功后，路径 /usr/local/go/bin 应该已经被添加到了 PATH 环境变量中，如果没有的话，你可以手动添加，和上面 Linux 的方式一样，把如下内容添加到 /etc/profile 或者 $HOME/.profile 文件保存即可。

```shell
export PATH=$PATH:/usr/local/go/bin
```

### 安装测试

以上都安装成功后，你可以打开终端或者命令提示符，输入 go version 来验证 Go 语言开发工具包是否安装成功。如果成功的话，会打印出 Go 语言的版本和系统信息，如下所示：

```shell
D:\code_work\go_learning>go version
go version go1.16.12 windows/amd64
```

## 环境变量设置

Go 语言开发工具包安装好之后，它的开发环境还没有完全搭建完成，因为还有三个重要的环境变量没有设置，它们分别是 GOROOT,GOPATH 和 GOBIN。

GOROOT：Go 语言安装根目录的路径，也就是 GO 语言的安装路径。

GOPATH：代表 Go 语言项目的工作目录，在 Go Module 模式之前非常重要，现在基本上用来存放使用 go get 命令获取的项目。

GOBIN：代表 Go 编译生成的程序的安装目录，比如通过 go install 命令，会把生成的 Go 程序安装到 GOBIN 目录下，以供你在终端使用。

假设工作目录为 D:\code_work\go_learning，你需要把 GOPATH 环境变量设置为 D:\code_work\go_learning，把 GOBIN 环境变量设置为 $GOPATH/bin。

在 Linux 和 macOS 下，把以下内容添加到 /etc/profile 或者 $HOME/.profile 文件保存即可。

```shell
export GOPATH=D:\code_work\go_learning
export GOBIN=$GOPATH/bin
```

在 Windows 操作系统中，则通过控制面板 -> 高级 -> 环境变量选项添加这两个环境变量即可。

在 Windows 操作系统中，则通过控制面板 -> 高级 -> 环境变量选项添加这两个环境变量即可。

## Go 编辑器推荐

好的编辑器可以提高开发的效率，这里我推荐两款目前最流行的编辑器。

第一款是 Visual Studio Code + Go 扩展插件，可以让你非常高效地开发，通过官方网站 https://code.visualstudio.com/ 下载使用。

第二款是老牌 IDE 公司 JetBrains 推出的 Goland，所有插件已经全部集成，更容易上手，并且功能强大，新手老手都适合，你可以通过官方网站 https://www.jetbrains.com/go/ 下载使用。

## 基础类型

常用的有：整型、浮点数、布尔型和字符串

###  整型

在 Go 语言中，整型分为：

- 有符号整型：如 int、int8、int16、int32 和 int64。

- 无符号整型：如 uint、uint8、uint16、uint32 和 uint64。

### 浮点数

浮点数就代表现实中的小数。Go 语言提供了两种精度的浮点数，分别是 float32 和 float64。项目中最常用的是 float64，因为它的精度高，浮点计算的结果相比 float32 误差会更小。

###  布尔型

一个布尔型的值只有两种：true 和 false，它们代表现实中的“是”和“否”。它们的值会经常被用于一些判断中，比如 if 语句（以后的课时会详细介绍）等。Go 语言中的布尔型使用关键字 bool 定义。

布尔值可以用于一元操作符 !，表示逻辑非的意思，也可以用于二元操作符 &&、||，它们分别表示逻辑和、逻辑或。

### 字符串

Go 语言中的字符串可以表示为任意的数据，比如以下代码，在 Go 语言中，字符串通过类型 string 声明：



###  零值
零值其实就是一个变量的默认值，在 Go 语言中，如果我们声明了一个变量，但是没有对其进行初始化，那么 Go 语言会自动初始化其值为对应类型的零值。比如数字类的零值是 0，布尔型的零值是 false，字符串的零值是 "" 空字符串等。

## 变量

Go 语言提供了变量的简短声明 :=，结构如下：

**变量名:=表达式**

## 指针

在 Go 语言中，指针对应的是变量在内存中的存储位置，也就说指针的值就是变量的内存地址。通过 & 可以获取一个变量的地址，也就是指针。

```go
pi:=&i
fmt.Println(*pi)
```

##  常量

常量的定义和变量类似，只不过它的关键字是 const。

##  iota

ota 是一个常量生成器，它可以用来初始化相似规则的常量，避免重复的初始化。假设我们要定义 one、two、three 和 four 四个常量，对应的值分别是 1、2、3 和 4，如果不使用 iota，则需要按照如下代码的方式定义：

```go
const(
    one = iota+1
    two
    three
    four
)
```

[beego教程](https://beego.vip/docs/install/env.md)

[go入门教程](https://laravelacademy.org/books/golang-tutorials)