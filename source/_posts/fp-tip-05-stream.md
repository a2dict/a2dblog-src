---
title: "FP05: 流式编程"
date: 2019-06-09 12:24:50
tags: fp
---

最近刚啃了遍《SICP》，想试试用JS纯FP实现流式编程。
流式编程一种介于函数式编程与命令式编程之间的编程范式，它把**状态**看做关于时间t的流函数`y=x(t)`，这样就可以用纯函数处理状态的变化。

Stream的概念和List很相似，但是Stream是状态关于时间的函数，所以理论上允许Stream的长度是无穷的。List不具备处理无穷的能力，所以需要引入`惰性计算`，定义无穷流。

```js
// cons构件
// let pair = cons(3, 5)
// car(pair) == 3
// cdr(pair) == 5
const cons = (a, b) => m => m(a, b)
const car = pair => pair((a, _) => a)
const cdr = pair => pair((_, b) => b)
const empty = cons()

// 延时计算
// force(delay(3)) == 3
// 由于js不支持macro，所以下面`delay(x)`直接写成`()=>x`， `force(()=>3) == 3`
const delay = obj => () => obj
const force = delayed => delayed()
// 流的头部，即s[0]
const head = car
// 流的尾部，即s[1:]
const tail = s => force(cdr(s))
// 流的第n位，即nth(s,n) == s[n]
const nth = (s, n) => n == 0 ? head(s) : nth(tail(s), n - 1)
// 取流的前n位
const take_mid = (rec, s, n) => n <= 0 ? rec : take_mid(rec.concat(head(s)), tail(s), n - 1)
const take = (s, n) => take_mid([], s, n)

// map
const stream_map = (s, m) => cons(m(head(s)), () => stream_map(tail(s), m))
// filter
const stream_filter = (s, pred) => pred(head(s)) ? cons(head(s), () => stream_filter(tail(s), pred)) : stream_filter(tail(s), pred)

// 
const stream_add = (s1, s2) => cons(head(s1) + head(s2), () => stream_add(tail(s1), tail(s2)))
// [1,1,1,1 ...]
const ones = cons(1, () => ones)
// [1,2,3,4 ...]
const integers = cons(1, () => stream_add(integers, ones))
// [2,4,6,8 ...]
const evens = stream_map(integers, x => 2*x)
// [0,1,1,2,3,4,7 ...]
const fibs = cons(0, () => cons(1, () => stream_add(fibs, tail(fibs))))
console.log(take(fibs, 10))
```
I hate js.

