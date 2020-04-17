---
title: APL Learning
date: 2020-04-17 16:47:48
tags: [编程, 兴趣]
---

被APL代码的希腊符号吸引了，作为一个不折腾会死星人，入坑了APL。

> APL is easy to learn in the sense that it's easy to get started. You'll be able to do simple things almost immediately.
> ... 
> If you haven't programmed before, none of these questions will bother you. You'll accept the way APL does things as natural and convenient. For this reason, APL has traditionally been used by people who are not primarily computer programmers, but who need to write quite sophisticated programs in the course of their work or research - actuaries, engineers, statisticians, biologists, financial analysts, market researchers, and so on. [1]

没错没错，确实挺简单的，而且看着X格还很高(斜眼笑.jpg)

<!-- more -->

# Intro
<img src="./apl-base.png">

最最最最简单的例子 `4 5 6 + 1 1 1`, APL以数组为单位，代码是从右住左读的(和Factor Lang一样)。上面这个例子结果是 `5 6 7`

由于APL代码是从右往左读的，所以`5 - 6 - 3 = 2`，而不是`-4`。


APL很显眼的一个特点是**代码使用所有unicode字符**。所以如果你愿意，也可以使用中文进行编程。
在APL中，通用的操作符有30+个（比任何编程语言都少得多），每个操作符都有对应的语义，相当于其它编程语言中的*函数*。

就像其它语言可以定义函数，用户也可以**定义操作符语义**。

常用操作符语义，请出门左转 wikipedia（我就是这么边查边学的，逃..

# An Example

求小于R的所有质数
```apl
(~T∊T∘.×T)/T←1↓⍳R
```

解析如下：
1、 `⍳R` 生成一个小于R的自然数数组, eg: [1 2 .. R]
2、 `1↓` 丢弃第一位，即 [2 3 .. R]
3、 `T←` 赋值给T
4、 `/` apl中的`/`不是除法，表示对数组中每一项执行。如果左边是数组的话，做过滤操作。eg: ` 1 0 1/ 7 8 9` => `7 9`
5、`T∘.×T` 矩阵乘法。结果是T中任意两项乘积的矩阵
6、`T∊` 判定T中元素是否在乘积矩阵中（非质数）
7、`~` 取反


# Sum Up

总的来说APL是门很有趣的语言。
做APL编程更像是在做智力题

1) 需要把握问题的算法过程
2) 将它转换面数据变换操作
3) 再选择合适的操作符组织成代码

PS: 某友吐槽学这个有什么用？好吧，确实没用。
不只APL，很多东西都没用！！对于很多人来说，吃饭活着就是一切了 无奈

有些事情，就算没用也要做，要不，人生也太无趣了。


# 参考：

[1] microapl-introduction http://microapl.com/APL/introduction_chapter1.html
[2] apl-programming-language.pdf http://courses.cs.vt.edu/~cs5314/Lang-Paper-Presentation/Papers/HoldPapers/APL.pdf
[2] apl-symbols-wiki https://en.wikipedia.org/wiki/APL_syntax_and_symbols 