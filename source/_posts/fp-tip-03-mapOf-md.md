---
title: "FP Tip03：Immutable Map 的纯FP实现"
date: 2019-03-11 15:01:01
tags: 编程
---

Map<K,V> 是最通用的容器类，几乎所有编程语言都将其加入标准类库中。
在纯函数式编程里，不允许副作用(side-effect)，不允许**状态**，只允许值与函数(btw: 函数也是值)。
FP是图灵完备的，可以使用函数模拟**状态**，实现Map容器。

<!-- more -->


```js
// 扩展partition函数
const last = xs => xs.length > 0 ? xs[xs.length - 1] : null
const partition = (xs, n) => xs.reduce((l, x) => {
    let t = last(l)
    if (t && t.length < n) {
        t.push(x)
    } else {
        l.push([x])
    }
    return l;
}, [])
Array.prototype.partition = function (n) {
    return partition(this, n)
}
// [1,2,3,4].partition(2) => [[1,2],[3,4]]

// Map容器的纯FP实现
const pair = ([key, value]) => k => k == key ? value : null
const merge = (l, r) => k => r(k) || l(k)
const mapOf = (...xs) => xs.partition(2).map(pair).reduce(merge, pair([null, null]))

// 示例
let m = mapOf("abc", 23, "def", 45)
m("abc")    // => 23
m("def")    // => 45
m("xxx")    // => null
```

还可以在上面的定义上再扩展`put`函数
```js
const put = (m, k, v) => merge(m, pair([k, v]))

let m2 = put(m, "foo", "bar")
m2("foo")   // => "bar"
```

PS：由于js奇怪的真值表，以上merge方法有BUG，应该使用`typeof()`做条件判断。此处代码只做演示用。

