---
title: vagrant搭建k8s集群
date: 2019-03-08 22:14:12
tags:
---

工作需要，需要搭建k8s多节点集群做测试。
由于搭建k8s是个很繁琐的过程，所以我想到了vagrant——一个像管理docker镜像一样的虚拟机管理工具。
搜索一番，果真有不少k8s的vagrantfile项目。我最终选择了[rootsongjc/kubernetes-vagrant-centos-cluster](https://github.com/rootsongjc/kubernetes-vagrant-centos-cluster)，一个由国人编写的项目。
下面记录使用vagrant搭建k8s集群的过程。

// @Deprecated 由于官方文档写得特别详细，不在此重复了 :) 