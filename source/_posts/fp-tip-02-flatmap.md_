---
title: "FP Tip02：数据扁平化"
date: 2019-03-10 12:58:45
tags: 编程
---

假设某商品接口返回数据如下

<!-- more -->
```js
let data = {
    "data": {
        "categories":[{
            "categoryId": "A01",
            "categoryName": "食物",
            "subcategories":[
                {
                    "subcategoryId": "S001",
                    "subcategoryName": "零食",
                    "items": [
                        {
                            "itemId": "001",
                            "itemName": "坚果"
                        },
                        {
                            "itemId": "002",
                            "itemName": "薯片"
                        }
                    ]
                },
                {
                    "subcategoryId": "S002",
                    "subcategoryName": "饮料",
                    "items": [
                        {
                            "itemId": "001",
                            "itemName": "可乐"
                        },
                        {
                            "itemId": "002",
                            "itemName": "橙汁"
                        }
                    ]
                }
            ]
        },{
            "categoryId": "A02",
            "categoryName": "玩具",
            "subcategories":[
                {
                    "subcategoryId": "S001",
                    "subcategoryName": "电玩",
                    "items": [
                        {
                            "itemId": "001",
                            "itemName": "Switch"
                        },
                        {
                            "itemId": "002",
                            "itemName": "PS4 Pro"
                        }
                    ]
                },
                {
                    "subcategoryId": "S002",
                    "subcategoryName": "益智",
                    "items": [
                        {
                            "itemId": "001",
                            "itemName": "积木"
                        },
                        {
                            "itemId": "002",
                            "itemName": "象棋"
                        }
                    ]
                }
            ]
        }]
    },
    "errno": 0
}
```

要想列表展示，需要对其进行扁平化，即转化成以下格式：

```json
[
    {
        "categoryId": "A01",
        "categoryName": "食物",
        "subcategoryId": "S001",
        "subcategoryName": "零食",
        "itemId": "001",
        "itemName": "坚果"
    },
    {
        "categoryId": "A01",
        "categoryName": "食物",
        "subcategoryId": "S001",
        "subcategoryName": "零食",
        "itemId": "002",
        "itemName": "薯片"
    }
    // etc..
]
```

应该怎么做呢？`for..for..for..` *STOP!!*

从FP的视角看，可以通过构造一个匹配函数进行转化。

```js
const concat = (x, y) => x.concat(y)
const flatMap = (f, xs) => xs.map(f).reduce(concat, [])
// 由于js没有flatMap方法，所以先自己定义
Array.prototype.flatMap = function (f) {
    return flatMap(f, this)
}

data.data.categories.flatMap(ctg =>
    ctg.subcategories.flatMap(sctg =>
        sctg.items.map(it => {
            return {
                "categoryId": ctg.categoryId,
                "categoryName": ctg.categoryName,
                "subcategoryId": sctg.subcategoryId,
                "subcategoryName": sctg.subcategoryName,
                "itemId": it.itemId,
                "itemName": it.itemName
            };
        })
    ))

```

这种处理方式比多重for简单得多，不过多级flatMap心智负担也不小。
如果是Haskell的话，处理起来会优雅得多。
在Haskell中，List is Monad，Haskell为monad提供了 `<-` 语法糖，可以这样写
```hs
do
    ctg <- categories
    sctg <- subcategories ctg
    item <- items sctg
return Item()

-- desugar: (没错，就是上面js的对应hs写法)
-- categories >>= \ctg -> subcategories ctg >>= \sctg -> items sctg >>= \item -> Item()
```