---
title: Robot Framework极简教程
date: 2020-03-20 15:02:15
tags: 编程
---

这是一篇Robot Framework极简教程。

<!-- more -->

# 一 简介
Robot Framework是一个Python编写的自动化测试框架。它能批量运行测试案例，并生成报告文件。

RF提供了一种语义化的DSL，后缀名为`.robot`。RF会将**测试步骤**映射成对应的**python方法**执行。

在RF ATDD工作流中，开发人员提供服务基础库，与测试、业务人员共同完成测试案例设计。

```robot
*** Settings ***
Documentation    极简的测试Robot

# 引入Python Library
Library     Collections

*** Variables ***
# 变量不区分大小写，但Variables块声明变量一般采用大写
# 文本
${NAME}     textValue
# 数组。对应 Python List
@{L1}       aaa  bbb  ccc  ddd
# Dict，即 Go Map
&{D1}       a=1  b=2  foo=bar

*** Test Cases ***
Log Something
    # 测试步骤
    Log  ${NAME}
    # 注意这里是`$`
    Log  ${L1}
    Log  ${D1}

Log All
    Log Many   ${NAME}  ${L1}  ${D1}

```

将上面代码保存为`Simple.robot`，运行`robot Simple.robot`即可查看测试结果。

# 二 基础语法

## 2.1 变量
变量在变量块中定义，不区分大小写，但声明中一般采用大写。

| var     | type | desc                                   |
| ------- | ---- | -------------------------------------- |
| ${NAME} | str  | 字符串。但python会在调用处自动类型转换 |
| @{L1}   | list | 数组                                   |
| &{D1}   | dict | 也变量go map                           | 

引用变量时，`${D1}`是直接引用；`&{D1}`是解构。


## 2.2 测试案例
每个测试步骤都有具体的**python方法**或**keyword**与之对应。

比如`Log Many`对应BuildIn里的`log_many()`方法。

开发人员可以方便地自定义Python方法扩展语义。

# 三 一些坑

## 3.1 空格
单空格在robot里是连接符；两个以上空格是分割符
```
@{L1}       aaa bbb ccc ddd
@{L2}       aaa  bbb  ccc  ddd

```
以上对应的Python代码是
```
L1 = ["aaa bbb ccc ddd"]
L2 = ["aaa", "bbb", "ccc", ddd"]
```
而且由于缺少格式化工具，很容易写错。
可以使用编辑器的正则表达式搜索`\s{2,}`高亮显示
