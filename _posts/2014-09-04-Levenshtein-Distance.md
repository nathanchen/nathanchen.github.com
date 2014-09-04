---
layout: post
title: "Levenshtein Distance"
category: 
tags: ["读文章","算法"]
---
{% include JB/setup %}

![img/btree1.png](/img/btree1.png)

**编辑距离**，又称Levenshtein距离，是值两个字串之间，由一个转成另一个所需的最少编辑操作次数，如果它们的距离越大，说明它们越是不同。

	如果str1="ivan"，str2="ivan"，那么经过计算后等于 0。没有经过转换。相似度=1-0/Math.Max(str1.length,str2.length)=1
	
	如果str1="ivan1"，str2="ivan2"，那么经过计算后等于1。str1的"1"转换"2"，转换了一个字符，所以距离是1，相似度=1-1/Math.Max(str1.length,str2.length)=0.8
	
##### 算法过程

	1、str1或str2的长度为0， 返回另一个字符串的长度。
	2、初始化（n + 1） * (m + 1)的矩阵d，并让第一行和列的值从0开始增长
	3、扫描两字符串（n * m），如果：str1[i] == str2[j]，用temp记录它为0；否则temp记为1。然后在矩阵d[i, j]赋予d[i-1,j]+1 、d[i,j-1]+1、d[i-1,j-1]+temp三者的最小值
	4、扫描完后，返回矩阵的最后一个值d[n][m]即是他们的距离
	
###### 第一行和第一列的值从0开始增长
	
![img/Levenshtein1.png](/img/Levenshtein1.png)

###### i列值的产生Matrix[i - 1, j] + 1 ; Matrix[i, j - 1] + 1   ;    Matrix[i - 1, j - 1] + t

![img/Levenshtein2.png](/img/Levenshtein2.png)

###### 以此类推直到矩阵全部生成

![img/Levenshtein3.png](/img/Levenshtein3.png)


### Reference

http://www.cnblogs.com/ivanyb/archive/2011/11/25/2263356.html
