---
title: Go重构指南
date: 2019-12-28 13:17:39
tags: 编程
---

我有代码洁癖，想把代码写得简单易懂。所以把我的重构经验写这吧。

<!-- more -->

# 定义

**重构（名词）**： 对软件内部结构的一种调整，目的是在不改变软件可观察行为的前提下，提高其可理解性，降低其修改成本。
**重构（动词）**： 使用一系列重构手法，在不改变软件可观察行为的前提下，调整其结构。

在过去，很多人使用重构这个词来指代码清理。其实，重构的关键在于**运用大量微小且保持软件行为**的步骤，一步步达成大规模的修改。每个单独的重构要么很小，要么由若干小步骤组合而成。因此，如果有人说他们的代码在重构过程中有几天甚至更长时间不可用，基本上你可以确定，他在做的事情不是重构。


# 原则

## 1 删除无效代码

服务版本升级，过渡阶段留下的代码，升级完成应该删除。
多余的代码不是资产！是负债！！

```go
if version == "v2" {
    // doSomething()
}
```

## 2 构建函数抽象

如果一个函数需要查看源码才知道干了什么，那这个函数没有存在必要。

## 3 构建数据抽象
假设系统用`"20:30:24|1001|3443"`表示「用户uid:1001在20点30分观看了id:3443」。
虽然这条数据很容易人肉解析，但编程中额外考虑实现细节是一种智力负担，应该尽量避免。
可以如下构建数据抽象
```go
type WatchEvent struct {
    Date    string
    Uid     string
    MediaID string
}
func (e *WatchEvent) Serialize() string {
    return fmt.Sprintf("%s|%s|%s", e.Date, e.Uid, e.MediaID)
}
func ParseWatchEvent(s string ) *WatchEvent {
    return &WatchEvent{}  // SKIP. you know it!!
}
```

## 4 保持不变性

1. 尽量不修改变量的值
2. 延迟变量定义。只在需要时才定义，并且定义后尽量不修改
3. 尽量不写有Side-Effect的方法（eg: 修改参数值、改变全局状态）；带Side-Effect的方法，应该明确标记。如果能像rust的强制**可变引用**就更好了

## 5 多用func，少用struct

纯函数比带有状态的结构更容易理解。
当一个方法引用多个Struct时，看代码人肉debug太烧脑。

## 6 函数只传值

函数尽量不传Client、Conn... 有副作用。
若需要传递Client，可以考虑构造struct抽象
```go
func UpdateSomething(conn *gorm.DB, t Something)
// =>
type ConnWrap struct {
    conn *gorm.DB
}
func (w *ConnWrap) UpdateSomething(t Something)
```


# 技巧

## 1 多重key Map可以使用KeyType

```go

var a map[string]map[string]int32

// => 
type KeyType struct {
    TagId string
    AllyId string
}
var b map[KeyType]int32

```


# 工具篇

放弃vim吧，拥抱IDE

1. 提取变量
2. 提取方法
3. 移动方法
4. 
