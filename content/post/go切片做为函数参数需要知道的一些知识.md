---
title: "go切片做为函数参数需要知道的一些知识"
description: "go切片做为函数参数需要知道的一些知识"
keywords: "golang"
date: 2022-08-21T22:09:00+08:00
lastmod: 2022-08-21T22:09:00+08:00

categories:
  - 笔记
tags:
  - golang

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

在 go 语言中,所有类型都是值传递, 这个知识在几乎所有 go 语法书中都有介绍. 但是切片和 map 这俩数据结构在做为函数参数的时候,可以通过形参改变实参. 这和值传递理念是相背离的.

这里做个小笔记增强记忆.

# slice 是一个结构体, 其中有一个字段是指向底层数组的指针

但是 slice 依然是值拷贝.
参考下面代码比较差异

```go
package main

import "fmt"

func myAppend(s []int) []int {
	s = append(s, 100)
	return s
}

func myAppend2(s []int) {
	s[0] = 666
}

func myAppendPtr(s *[]int) {
	*s = append(*s, 100)
}

func main() {
	s := []int{1, 1, 1}
	myAppend(s)
	fmt.Println(s)
	myAppend2(s)
	fmt.Println(s)
	myAppendPtr(&s)
	fmt.Println(s)
}

// [1 1 1]
// [666 1 1]
// [666 1 1 100]

```
