---
title: 停机问题
date: 2020-08-09 23:26:56
tags: 科学
---


## 一个证明

对停机问题最简洁的证明，来自《The Little Schemer》——一本少儿编程启蒙书。

```scheme
;; eternity
(define eternity
  (lambda (x) (eternity x)))

;; 假设存在函数`will-stop?`，它能判断一个程序是否会导致停机
(define last-try
  (lambda (x)
    (and (will-stop? last-try)
         (eternity x))))

;; 这代码行为如何？
(will-stop? last-try)
```

## 背后的数学

### 数学的处理方法: 同构
程序 同构于 符号系统 同构于 数论系统（Peano公理系统）

### Peano公理系统
### 哥德尔不完备性定理

#### ω-不完备

#### Peano公理系统不完备

### 结论：不可计算性
