---
title: "FP技巧01：正则表式处理字符串"
date: 2019-03-09 18:56:23
tags: fp
---

在纯函数式编程(pure fp)里，不允许副作用，所以对于字符串处理，常常需要使用regex转换成数据流，再对数据流进行操作。

以下为js代码

```js
// 1.驼峰转下划线
let camelcaseToUnderscore = s => s.split(/(?=[A-Z])/).join('_').toLowerCase()
camelcaseToUnderscore('abcDefGhi')    // => "abc_def_ghi"

// 2.下划线转驼峰
let capitalize = s => s.replace(/^./, s[0].toUpperCase())
let underscoreToCamelcase = s => s.split('_').map(capitalize).join('')
underscoreToCamelcase('abc_def_gh') // => AbcDefGh

// 3.字符向量化
'aaabbbbaaaccddd'.match(/(\w)+?(?!\1)/g).map(it => ''+it[0]+it.length).join('') //"a3b4a3c2d3"

```