---
layout:     post
title:      Levenshtein distance之python实现
category: tech
description: 当我们在用搜索引擎(google，百度)，或者有道字典时，如果我们的输错了关键字时......
tags: python, 算法
---

当我们在用搜索引擎(google，百度)，或者有道字典时，如果我们的输错了关键字时，系统会自动提示我们"你找的是不是XXX"。那么这个在原理上面是怎么实现的呢？

其实它就是一个计算字符串编辑距离算法的应用。系统去查找与输错的关键字A距离最短的那些搜索词，然后提示你是不是这些搜索词。

##Levenshtein distance

最基础的就是[Levenshtein distance](http://en.wikipedia.org/wiki/Levenshtein_distance)，Levenshtein定义一次删除、插入、单字符替换的操作为1。

其DP公式为：

![Levenshtein Distance DP公式](/images/2012/LevenshteinDP.jpg)

可以注意到Levenshtein Distance算法没有考虑交换的操作，比如AC-->CA，如果用Levenshtein来计算的话L(AC, CA) = 2。但是按照常识AC-->CA的距离应该只是1而已，因为A与C只需要进行一次交换的操作就达到目的了。

##Damerau-Levenshtein distance

所以Levenshtein与Damerau一起又改进了原来的Leveshtein distance算法，发明了[Damerau-Levenshtein distance](http://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance)

Damerau-Levenshtein算法分两种情况，第一种比较简单，称为Optimal string alignment。这种情况在交换的字符之间没法插入其他字符。比如OSA(CA, ABC)=3，它的具体操作是 CA -> A -> AB -> ABC 。

OSA这种情况只需要对原来的Levenshtein算法进行一小部分的改进，只需要添加如下一段代码就行了：

	if i>1 and j>1 and source[i]==target[j-1] and source[i-1]==target[j]:
		d[i, j] = min(d[i, j], d[i-2, j-2]+cost)

但是明显的CA与ABC的转换只需要两次操作就行了，CA -> AC -> ABC。 即LD(CA, ABC)=2，这就是第二种情况

第二种情况比较复杂，具体的python代码如下：

<script src="https://gist.github.com/3799352.js">
</script>

##其他算法

其他的求编辑距离的算法还有：Jaro-Winkler, n-gram（基于概率模型的）。具体的实现原理我暂时还没有去了解过，所以也就点到为止了。

##Tips

关于纠错功能的提示，可以维护一个常用的相似度最高的几个词的字典。至于何时启用纠错功能，我觉得可以在搜索结果少于XX条时，或者热度少于XX时提供纠错功能。
