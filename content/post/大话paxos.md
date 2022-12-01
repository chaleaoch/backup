---
title: "大话paxos"
description: "大话paxos"
keywords: "paxos"
date: 2022-12-03T13:49:00+08:00
lastmod: 2022-12-03T13:49:00+08:00

categories:
  - 笔记
tags:
  - 理论派

# Post's origin author name
#author:
# Post's origin link URL
#link:
# Image source link that will use in open graph and twitter card
#imgs:
# Expand content on the home page
#expand: true
# It's means that will redirecting to external links
#extlink:
# Switch to enabled or disabled comment plugins in this post
#comment:
# enable: false
# Enable table of content
toc: true
# Absolute link for visit
#url: "{{ lower .Name }}.html"
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
---

一篇笔记. 不够严谨.

paxos 是一个**共识**算法, 大白话理解就是一个系统中的所有节点, 对一件事达成共识. 这件事可能是一个动作, 譬如 `+1`, 也可能是一个值 譬如 `set key=value`
应用场景:
有一组状态机, 譬如 redis. 对 redis 进行一系列的操作 `[+1,-1,+1,-1,+1,-1,+1,-1,+1,-1,+1,-1,...]`
我们要保证 redis 集群中的所有节点,收到的消息的顺序是一致的.

容错模型:

- 非拜占庭网络, 也就是说没有黑客恶意修改数据.
- 允许节点损坏和网络延迟.

paxos 中的角色:

- proposer, 提出提案(一件事)的人. 下图中的吴凡和李峰.
- acceptor, 做出决策的人, 很多 proposer 会提交很多提案, 但是只有一个提案会被接受(只有这样才可能达成**共识**). acceptor 就是那个做出决策的人(们)., 下图中的中间三个人,冯刚,张某和陈哥.
- learner, 不提出提案, 只参与学习. 类似粉丝. 下图没有体现.

提示: 同一个节点可以兼任上面三个角色, 也就是说, 一个节点既可以提出提案,也可以做出决策.

大话一个例子:
吴凡和李峰分别提出了提案. 最后所有人达成共识. 下图只是一种比较典型的时序场景, 实际场景远比下图要复杂的多. 精力有限无法全部画出.
最后的学习阶段比较简单. 就是当一轮决策产生之后,learner 向 accepter 学习一下就行了. 这里不是重点.
![](https://fastly.jsdelivr.net/gh/chaleaoch/CDN@main/images/1669866959708paxos.jpg)
