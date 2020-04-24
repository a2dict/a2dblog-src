---
title: I'm Clojurian
tags:
  - 编程
  - 随笔
date: 2020-04-17 23:36:47
---


> 写clojure吧。
> 其它语言写不好，可以骂xxx傻逼；clojure写不好只能骂自己傻逼。
> clojure至少可以帮人认清事实 :)

<!-- more -->

从我初次体验编程(VB)至今，已有10多个年头。
折腾过不少东西，脚本编程、GUI、反汇编、工程计算与分析... 
很庆幸这么多年都过去了，仍保留着对计算的一丢丢热情。
Clojure对我编程方法论影响很大，是一项很有趣的技术。

# Clojure 简介

Clojure是一门运行在JVM上的LISP方言。
LISP是公理化的编程语言，它提供了基础编程原语（七大公理），可以演绎一切计算；它没有语法，代码与数据具有相同的结构——列表，可以像数据一样处理代码。

clojure通过macro像操作数据一样操作代码。我最喜欢的一个例子
```clojure
; Clojure S-表达式，操作符前置
(* 3 5) ; => 15

; 如果喜欢操作符中置写法，可以通过macro实现
(defmacro calc [a op b]
  (list op a b))
(calc 3 * 5) ; => 15
```
clojure采用S-表达式。*语句*是一个列表，列表的第一项是操作符/函数。
宏(macro)是一类特殊的函数，它将clojure代码当成参数，并生成clojure代码。
由于宏的存在，clojure提供了一种更灵活组织代码的方式——你可以用任何愿意的格式组织代码，关键在于编写一个正确的macro。

上面举这个例子，有两个目的。
1. clojure macro有操作代码的能力
2. 通过macro，可以扩展自定义语法

由于语法的灵活性，clojure颠覆了传统编程方法——clojure解决问题的方式是逆向的。


# Clojure 编程方法论

## 数据与代码

什么是数据，什么是代码？
很多人开发中并不明确区分两者的边界。

举个粟子。
存储在数据库中的是数据；与数据库建立连接、读取数据的是代码。
配置文件是数据；而解析配置文件，启动程序是代码。

如果说，**算法 = 控制 + 逻辑**。「代码」就是程序的控制部分，「数据」即逻辑。

编程、设计、架构本质上都是在做着同样的事——分离「控制」与「逻辑」，用数据表达逻辑。
因为，数据总比代码更容易理解，更容易修改。

## 数据即代码、代码即数据

如果代码与数据具有相同的格式，会怎样？
这就是LISP家族的「代码即数据」。

在技术上，clojure的代码就是一个列表，很容易进行代码分析。
运用技术方法从代码中发现**模式**，并构建抽象，可以写出简洁的代码，语言上限很高。

在心理上，当程序员可以像操作数据一样操作代码，代码的语法不再重要（LISP没有语法，或者说可以自己定义语法）。
这个特点迫使程序员思考，如何组织代码以表达问题？让代码迁就问题，而不是问题迁就代码。
Clojure思考问题的方式是逆向的。

这是一个web-router，它定义了一个`GET`请求返回`Hello World`的web服务。
```clojure
(defroutes app
  (GET "/" [] "<h1>Hello World</h1>")
  (route/not-found "<h1>Page not found</h1>"))
```

同样的功能，用Go实现
```go
func main()  {
	mux := http.NewServeMux()
	mux.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("<h1>Hello World</h1>"))
	})
	http.ListenAndServe(":8080",mux)
}
```
Go自带了HTTP标准库，所以实现起来还不算太复杂。
Clojure版本是*声明式*的，它是一个嵌套的数组结构，描述路由的行为；
而Go版本是*命令式*的，你需要告诉它，如何创建`mux`，以及`Write` `[]byte`，这些都是与问题域无关的「控制」。

控制并不是好东西，它会诱导开发者迷失在API细节中，而无法专注于问题本身。

一般程序员思考问题，关注点在于代码。从API出发，通过语言、框架提供的API组织用于解决问题的代码；
而Clojure(LISP)程序员，关注点在于问题本身。从问题出发，定义「描述问题的结构」，通过macro生成目标代码。

## 其它语言不能做到吗？

当然可以。*这个方法论*同样适用于其它任何一门图灵完备的编程语言。

应用同样的方法——数据驱动程序，先定义用于描述问题域的数据格式（通常是json或yaml），将数据保存在文件系统或etcd中；再编写一个Parser，构建起数据到程序的桥梁。
*（没错，编程就是人肉Parser！！*

这种技术处理在工程应用中十分普遍。
程序员都是*聪明人*，很多人无意识中学会这种构建程序的方式。

下次开始一个程序的设计时，先思考下，如何用数据**精确**地描述问题域的**复杂性**吧。

# Clojure 独占

TBC