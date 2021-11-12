---
title:       "go-zero框架热编译"
subtitle:    ""
description: ""
date:        2021-10-29T10:38:45+08:00
author:      "pengjl"
image:       ""
tags:        ["golang","微服务"]
categories:  ["go-zero" ]
---

#### 前言

这段时间开始把现有业务逐步往微服务架构迁移，框架这块选的go-zero，出自于晓黑板，框架本身是不支持热编译的，导致在开发过程中改动代码后需要停止，然后重新运行，大大降低开发效率。

刚开始学golang的时候，选的beego作为入门框架，现有业务也是用beego写的，自带的bee工具支持热编译，在切换到go-zero的时候就很不习惯了，开始琢磨要让go-zero也自动编译。

我们在刚接触golang的时候，要启动一个go程序，通常就是命令：
``` shell
go run xxx.go
``` 
beego在启动项目时，使用 `bee run`,它去找main包，main方法，并编译一个可执行文件，默认是执行目录文件名，在当前目录下任意文件被修改保存后，都将重新编译，达到自动编译的目的。

go-zero通过goctl工具生成的api代码，同样存在main包，main方法，是否可以尝试试一下，反正也不花钱。

#### 方法一
如下图所示，在go-zero的服务目录中执行`bee run` 命令，正常启动，并且生成一个与根目录同名的可执行文件，结合postman调试流程如下图所示：

![avatar](/img/2021-10-29-02.png)

结果是正常的，没什么问题，

#### 方法二

在网上也看到其他的工具能够实现。例如 `rizla` 源于另一个go web框架 `Iris` 使用方法如下：

```shell
#安装 rizla 工具
go get -u github.com/kataras/rizla

# 启动项目
rizla main.go
```

#### 总结

- go-zero框架开发过程中实现热编译，最少有两种方法：
  - beego 框架的bee 工具。
    ``` shell
    #安装
    go get -u github.com/beego/bee/v2
    #运行
    bee run
  - Iris 框架的 rizla 工具，使用法方法不再重复描述了。

由于目前只接触过 `beego` 和 `go-zero` 这两个框架，一个支持热编译，一个不支持热编译，可以推断其他不支持热编译的框架都能够通过支持热编译的工具来启动。