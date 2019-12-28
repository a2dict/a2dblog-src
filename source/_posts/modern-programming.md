---
title: 现代编程
date: 2019-11-30 16:51:23
tags: 编程
---

我一直觉得自己在程序员中是个怪人。相比于研究框架、中间件，钻研网络、JVM等技术细节，我对编程语言更感兴趣。

编程从形式上看是**plain-text model world**。就是使用代码（纯文本）对现实问题进行建模，进而用计算模拟现实。
编程语言代表着对问题的抽象和思考方式。相比于计算细节，我对“如何思考问题”更感兴趣点。

直到认识松本行弘（Ruby作者，看过他的书）。

> 从年轻的时候开始，我就对编程语言有着极为浓厚的兴趣。比起“使用计算机干什么”这一问题，我总是一门心思想着“如何将自己的意图传达给计算机”。从这个意义上说，我认为自己是个“怪人”。但是，想选择一个能让自己的工作变得轻松的编程语言，想编写一种让人用起来感到快乐的编程语言，一直是我梦寐以求的，这种迫切的心情恐怕不输于任何人。

受Paul Graham影响，入坑了Lisp。并把学习PL当作一项爱好，保持一年至少学习一门新语言。
从工作至今，业余学习过的编程语言有：Python、Typescript、Haskell、Clojure、Kotlin、Go、Rust，以及现在边学边用着的Scala2.13。

在学习上有一个悖论：学习得越多，越发觉自己无知。但越容易接受不确定性。

学习PL的好处是
1) 从PL看所有编程语言都是某些特性的组合。编程语言学得多了更容易理解语言的设计初衷，进而更容易掌握新的编程语言
2) 当遇到相同的问题，容易跳出圈子用新的方式思考问题（OOP、FP、面向表达式编程、面向数据编程、Monad...）

我发现monoglot容易产生语言崇拜，偏爱某一语言而黑其它语言；
polyglot更容易从跳出语言的限制，做出更simple的设计。

软件工程上没有银弹，更没有一种适用于任何场景的编程语言。(注：原文指的是工程管理技术)

<!-- 
我一直觉得一个程序员至少掌握3门语言。
而且一般有一定工作经验的话，都不只限于任一语言吧。
- 工作的语言（Java、Rust、C/C++）。
- 脚本语言（Python、Ruby、Groovy）。用于快速原型设计和处理日常问题
- 快乐的编程语言（Clojure、Ruby）。用于探索编程并实现自己想法 -->


声明一下，我并不从事专业PL研究，只是以一个业余爱好者的视角分享我的PL心得。
如果以下内容有什么不准确的，烦请谅解指正。

在我看来，现代编程语言有两条发展主线
- Meta-Programming (元编程)
- Type-Level Programming

这两方向的代表分为Lisp和Haskell；对应的JVM语言分别为Clojure和Scala。

## Meta-Programming

实现Meta-Programming的方式主要有两种
- 宏。代码生成代码，一般是编译时
- 元数据驱动编程。通过给代码符加元数据（如Java注解），运行时动态修改代码行为

宏实现元编程的代表是Lisp; 元数据驱动编程的代表是Ruby。

Lisp宏的元编程能力最强，以至于得到“面向编程语言编程”的称号。
在Lisp中，代码与数据具有相同的结构（同像性），可以像操作数据一样操作代码，进而扩展自身语言能力。
(注：相比之下，其它语言只能通过语言升级)

```clojure
;; clojure
(+ 3 5) ;; => 8
;; 扩展中缀运算
(defmacro calc [a op b]
  (list op a b))
(calc
  3 + 5) ;; => 8
```
例1: Lisp的S表达式默认是前缀运算，如果愿意可以很容易通过Macro扩展出习惯上的中缀运算。

如果说编程形式上是用代码构建现实问题抽象，那宏就是用代码构建代码抽象。
编程中有个原则，叫DRY。对于重复的Statement人们抽象出Method；对于一组Method，集合成Package；尽量保持代码可复用。

某些场景由于编程语言能力的限制，会产生大量boilerplate code。
一般的解决方案是编写一个代码生成器，我相信很多程序员都写过类似的工具。

而宏提供了一种对代码进一步抽象的能力。
```clojure
;; 代码来源 http-kit
;; 定义http-request方法模板
(defmacro ^:private defreq [method]
  `(defn ~method
     ~(str "Issues an async HTTP " (str/upper-case method) " request. "
           "See `request` for details.")
     ~'{:arglists '([url & [opts callback]] [url & [callback]])}
     ~'[url & [s1 s2]]
     (if (or (instance? clojure.lang.MultiFn ~'s1) (fn? ~'s1) (keyword? ~'s1))
       (request {:url ~'url :method ~(keyword method)} ~'s1)
       (request (merge ~'s1 {:url ~'url :method ~(keyword method)}) ~'s2))))

(defreq get) ;;定义get方法
(defreq delete);;定义delete方法
(defreq head)
(defreq post)
(defreq put)
(defreq options)
(defreq patch)
(defreq propfind)
(defreq proppatch)
(defreq lock)
(defreq unlock)
(defreq report)
(defreq acl)
(defreq copy)
(defreq move)
```

其实上面代码还可以进一步简化，干掉重复`defreq`。
```clojure
(defmacro defreqs [& methods]
  (for [m methods]
    `(defreq ~m)))
(defreqs get delete head post put options patch propfind proppatch lock unlock report acl copy move)
```

像操作数据一样操作代码真的很酷。


## [TODO]Type-Level Programming
<!-- 
这篇一直很难下笔，因为，我也还在学习中，很难把Type说清楚。

按Haskell文档，最早是为了引入类型约束而引入type class，但是后来发现type class的作用超出了预期。
-->
未完。。
