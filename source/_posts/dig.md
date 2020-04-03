---
title: 重新认识控制反转
tags:
  - 编程
  - 设计
date: 2020-04-04 00:21:25
---

控制反转(IoC)、依赖注入(DI)、依赖反转原则(DIP)...
都是些什么东西？反转了什么？

<!-- more -->

> "The question is, what aspect of control are [they] inverting?" Martin Fowler posed this question about Inversion of Control (IoC) on his site in 2004. Fowler suggested renaming the principle to make it more self-explanatory and came up with Dependency Injection.
> —— [spring framework ioc](https://docs.spring.io/spring/docs/4.3.26.RELEASE/spring-framework-reference/htmlsingle/#overview-dependency-injection)

最早是在学习Spring的时候，知道了「控制反转」和「依赖注入」的概念。
Spring文档也没说清楚，大概意思是，「依赖注入」是「控制反转」的别名。

# 「控制反转」到底反转了什么？

<img src="./ioc-01.svg">

如上，是一个正向控制UML图。实线表示代码依赖，虚线表示控制。
A、B 的变更都可能会导致Main的变更，Main很不稳定。

通过引入接口，反转依赖。

<img src="./ioc-02.svg">

这样A依赖于Main，而Main不受A变更影响。

# 控制反转(IoC)、依赖注入(DI)、依赖反转原则(DIP)
IoC是面向对象的设计方法，常用的实现方式是DI。通过反转依赖方向，可以控制类、组件的「稳定性」。

依赖反转原则是类设计、组件设计、架构设计上常用的解耦手段。
