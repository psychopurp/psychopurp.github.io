---
title: 2021百度校招+后端开发面试
date: 2021-04-12 14:19:12
category: 面经
top: true
cover: true
summary: 2021 百度校招面试过程
keywords:
  - 百度面经
  - 后台开发
  - 面度面试
tags:
  - 面试
  - 后端开发
  - 百度
---

大四上学期一直在字节跳动实习，期间也没参加其他公司的秋招，今年四月份想再看看春招有没有更好的机会，前几天在一个 Golang 交流群里看到了一个百度的机会，看介绍是个 955，而且挺核心的一个技术团队，技术栈也是 go。于是直接给那个邮箱投了简历，第二天就给我打来了电话，约了下周一的面试。

---

# 一面 技术面（90 分钟左右）

一面让我下载了他们的工作平台：如流（类似飞书），使用此软件来进行视频面试。面试官很专业，问题主要分为项目，计算机基础，算法等部分，最后给了我面试评价，以及让我问了一些问题。

## 问题

最开始自我介绍，我大概介绍了下学校和专业，以及去年的实习经历和实习所做的一些工作。

### 1.项目相关

项目相关问题都是根据我实习工作来提问的

1. 你做项目遇到的最难的问题是什么？（以前没咋思考过这个问题，就说了一个做的比较复杂的项目）
2. rocket mq 选择某个组件详细介绍原理？（讲了 broker,producer,consumer 相关一些联系
3. rocket mq 如何保证消息传递？
4. consumer 能否知道自己消费到哪里了？
5. redis 基础数据结构？使用过哪些？有几种数据持久化方案？RDB 和 AOF 区别？

### 2.计算机基础

1. 进程和线程的关系？
2. 如何查找 CPU 占用资源高的进程？如何查看端口占用情况？
   > 讲了 linux 的 top，lsof -i，netstat -nltp 相关的命令。
3. 如何查找僵尸进程？
   > 这个记得有个 linux 命令可以查询，但是面试没想起来
4. TCP 三次握手是怎样进行的？
5. TCP 为什么是可靠的传输协议？拥塞控制介绍？
6. HTTP 常见状态码？
7. HTTP 和 HTTPS 有什么区别？HTTPS 的连接过程？是对称加密还是非对称加密？
   > https 的原理和区别回答出来了，但是 SSL 层认证的详细过程记得不清楚，大致讲了一下，结果最后不确定是对称加密还是非对称加密 😓
8. HTTP2.0 和 HTTP1.0 的区别？
   > 这两个区别好久前看过，回答的不太好。
9. 从输入网址到获得页面的过程？DNS 是怎么进行查询的？
   > DNS 讲了使用递归查询。记得也有点模糊了。
10. 如何实现分片下载和分片上传？
    > 还好之前自己写过分片下载，知道原理，这块回答的好。
11. 数据库索引引擎有哪些？有什么区别？
    > 数据库问的都是 mysql 相关的。讲了 innodb 和 myisam,和一些基本区别。
12. 数据库的四种隔离级别？如何实现可重复读？MVCC 介绍原理？
    > 隔离级别都讲出来了，实现可重复读回答了 mvcc，但 mvcc 原理没答出来。
13. 为什么索引引擎使用 B+树而不是二叉树？使用二叉树为什么会效率低？
    > B+树的话扁平 IO 次数少，而且命中索引后可以读入大块数据到内存，二叉树因为结构导致 IO 次数非常高。

### 3.算法

因为简历里有写过使用前缀树解决了一个实际业务问题，面试官就让我在线写一个前缀树。我就共享屏幕用 golang 写了一个，不过费了点时间。写完跑了测试并给他讲解了下思路。

#### 实现一个前缀树

```go
package main

import (
	"fmt"
)

type Node struct {
    Prefix int32
    Full   string
    IsEnd  bool
    Nodes  map[int32]*Node
}

type Trie struct {
	Root *Node
}

func (t *Trie) Insert(item string) {
	if t.Root == nil {
		t.Root = &Node{
			Prefix: int32(' '),
			Full:   "",
			IsEnd:  false,
			Nodes:  make(map[int32]*Node),
		}
	}
	node := t.Root
	for index, char := range item {
		// 如果存在则获取子节点
		if tmp, exist := node.Nodes[char]; exist {
			node = tmp
		} else {
			// 如果不存在则插入,并进入子节点
			node.Nodes[char] = &Node{
				Prefix: char,
				Full:   item[0 : index+1],
				IsEnd:  false,
				Nodes:  make(map[int32]*Node, 0),
			}
			// 如果是最后一个字符，则设置结束符号
			if index == len(item)-1 {
				node.Nodes[char].IsEnd = true
			}
			node = node.Nodes[char]
		}
	}
}

func (t *Trie) PrefixSearch(prefix string) []*Node {
	node := t.Root
	var result []*Node

	// 先找出该prefix 的子节点
	for _, char := range prefix {
		if tmp, exist := node.Nodes[char]; exist {
			node = tmp
		}
	}
	if node == t.Root {
		return result
	}

	DFS(node, &result)
	return result
}

// 找出该子节点下所有IsEnd 节点
func DFS(node *Node, result *[]*Node) {
	if node == nil {
		return
	}
	for _, n := range node.Nodes {
		if n.IsEnd {
			*result = append(*result, n)
		}
		DFS(n, result)
	}
}

func GenerateTrie(data []string) *Trie {

	trie := new(Trie)
	for _, v := range data {
		trie.Insert(v)
	}
	return trie
}

func main() {
	data := []string{"hello", "hex", "world", "war", "water"}
	trie := GenerateTrie(data)

	nodes := trie.PrefixSearch("w")

	for _, n := range nodes {
		fmt.Printf("%v ", n.Full)
	}
	fmt.Println("")

	nodes = trie.PrefixSearch("he")
	for _, n := range nodes {
		fmt.Printf("%v ", n.Full)
	}

	//fmt.Println("hello",nodes)
}

```

## 最后总结

最后面试官告诉我需要再看一下 https 加解密实现的流程，以及 mysql mvcc 如何实现可重复读隔离级别的相关内容。面试官很 nice，最后自己把 https 的加解密给我讲解了一下。

然后我提问了一些他们团队所做的事相关的问题就结束了

---

# 二面 技术面（70 分钟左右）

周一刚面完，周二 HR 就找我约了周三的面试，也是同一个时间点，因为一面让我下去再看看 https 和 mysql 的 mvcc 相关内容，我怕二面又来问我这个，就好好准备了这两块内容，但二面没有问基础内容，全程聊了项目和工程，后面是共享屏幕来介绍我的个人项目和算法代码，最后讲了两道 leetcode hard 算法题。

## 问题

二面问题基本围绕着我在简历上写的项目进行的，所以中间一些详细的问题就略过啦。

### 1.项目相关

1. 你做的项目是部署到了多少台机器？(这个应该是想问我做的项目的规模
2. 分布式系统中怎么实现幂等性？可靠性？
3. redis 熟悉吗？有哪些基本数据结构？
4. 你用 go 写过并发程序吗？要怎么做？
   > 这个应该是考察语言熟悉程度，回答了 sync.Mutex,sync.Map 之类的使用，以及并发程序需要注意的点。
5. 如何给 go net/http 实现的接口添加一个中间件？
6. 如果让你设计一个中间件的功能你会使用什么数据结构？你会怎么实现？
   > 之前学习 gin 框架 的时候模仿写过一个 web 框架，就介绍了这个项目中实现中间件的源码和思路。

### 2.算法

1. 你最近刷过哪些算法题？来讲讲几个？
   > 面试官应该是看了我的 github，刚好最近在研究两道 hard 级别的题，我就打开介绍了下我写的思路，优化的点，以及多种解决方法。

## 最后总结

- 二面是来考察我实际做项目的能力，以及我对语言的熟悉程度，很多点都是根据我在简历上写的经历来提问，比如：这么做是为了解决什么问题？如果 xxx，你怎么处理？有没有考虑 xxx 点？之类的。
- 二面基本没有失误，自己也感觉发挥的很好，期间问的一些问题没理解面试官意思，面试官也不急不躁给我讲解，后面我问问题的环节就问了一些关于工作环境，团队规模之类的。面试体验也很好，心想应该没问题了，果然过了两小时 HR 就来跟我约周四的第三次面试了。
