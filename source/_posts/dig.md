---
title: 重新认识控制反转
tags:
  - 编程
  - 设计
date: 2020-04-04 00:21:25
---

控制反转(IoC)、依赖注入(DI)、依赖反转原则(DIP)...
都是些什么东西？反转了啥？

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

# 控制反转(IoC)、依赖注入(DI)、依赖反转原则(DIP)是什么关系
IoC是一种面向对象设计方法，常用的实现方式是DI。通过反转依赖方向，可以组件解耦，并控制其「稳定性」。

DIP是一项设计原则，同时也是组件设计、架构设计上常用的解耦手段。

# 为什么需要DIP

做软件架构时，需要理清组件（服务）间的依赖关系。
在依赖关系中 `A -> B`意味着A需要跟随B的改变而变化。A的变化比B频繁。

在一个系统中，不同组件由于变更原因而变更速率不同。
软件架构中很重要的一项工作是，寻找组件的边界，隔离组件变化引起的副作用，并适应不同组件不同迭代速度。

DIP 提供了最基本的**操控组件依赖关系**的能力。
架构师可以按组件的变更速率，重新组织系统的依赖关系。

# DIP在架构设计上的应用

这里有一个经典问题。

一个项目采用分层架构，由Controller-Service-DAO组成。DAO接口应该放在哪？
<img src="./arch-01.svg">

这是正向控制流程，service 依赖于 DAO。
许多系统，数据层都很少变动，对数据层稳定性要求比服务更高，这方案并没有什么大问题。
但在微服务情境下，多个独立部署的服务共同依赖相同的数据层，会造成「共享数据耦合」，使得数据层越来越难被改动。若不警惕，最终会演化成为大泥团。

<img src="./arch-02.svg">

相反，如果把接口移到service层，服务中屏蔽了数据细节。
服务可以根据需要更换数据层实现，使用MockImpl实现服务层自动化测试，服务更加自洽。


PS: 架构设计从来就没有正确解。哪种方案更适合，需要具体场景具体分析。
BUT 很多时候，技术甚至不是问题的最优解。

