---
layout: post
title: "莎士比亚与无限猴子定理（一）"
tagline: "不懂得交流的猴子们"
description: ""
category: Math
tags: [Shakespeare, Monkeys, Python, Probability, Linear_regression, Maximum_likelihood]

---
{% include JB/setup %}

##引言
去年七、八月份，看的书、玩的游戏里频繁地出现了如下关键词：

> 莎士比亚、无限猴子定理、博尔赫斯、巴别图书馆、失控、世界尽头的图书馆、五彩斑斓的世界

作为一个有些迷信的人，我觉得这个上天的暗示太强烈了，我一定要研究研究这个问题= =于是= =试试做个无限猴子定理的实验。

翻出01年的旧机器（我才不想糟蹋新电脑呢= =而且实验的时候我还要玩游戏呢= =），跑了跑这个实验。

**其实最后让我做成了概率拟合的实验。**

##无限猴子定理

那，什么是无限猴子定理？[维基百科的“无限猴子定理”条目](http://zh.wikipedia.org/zh/無限猴子定理)给出了以下的简要定义：

>無限猴子定理的表述如下：让一只猴子在打字机上随机地按键，当按键时间达到无穷时，几乎必然能够打出任何给定的文字，比如莎士比亚的全套著作。
>
>在这里，几乎必然是一个有特定含义的数学术语，“猴子”也不是一只真正意义上的猴子，它被用来比喻成一个可以产生无限随机字母序列的抽象设备。这个理论说明把一个很大但有限的数看成无限的推论是错误的。猴子精确地通过键盘敲打出一部完整的作品比如说莎士比亚的哈姆雷特，在宇宙的生命周期中发生的概率也是极其低的，但並不是零。

也就是说，找个不会死的猴子和一个永远不会坏的打字机，以及无穷无尽的纸张，给打字机装上自动进纸系统，让这个猴子没日没夜地敲，等一万两千年（甚至更长时间= =）后，我们可以从数不过来的纸里挑出有限的几张，装订好，就是莎士比亚全集。

好蛋疼= =

##实验一：猴子单次实验

我给猴子的是一台可以敲出如下字符的打字机：

	abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ!"#$%&'()*+,-./:;<=>?

先让我们小试猴刀，让它试试打出`cat`这个单次需要多长时间。

以下，是我给本次实验准备的`Python`脚本，一切都是标准库的。

	import random
	import string
	from time import time
	
	charset = string.ascii_letters + string.punctuation + ' '
	destination = "cat"
	with open('report.txt', 'w') as fout:
		fout.write("char set: %s\n" % charset)
		fout.write("destination: %s\n" % destination)
		fout.write("All possible Permutations: %d\n" % len(charset) ** len(destination))
		counter = 1
		t = time()
		while 1:
			generated = "".join(random.choice(charset) for i in xrange(len(destination)))
			if generated == destination:
				break
			else:
				counter += 1
				if counter % 10000 == 0:
					random.seed()
					print generated
		t = time() - t
		fout.write("The monkey has been tried %d times, and cost %d h %d min %f s.\n" % (counter, int(t)/3600, (int(t)%3600)/60, t%60))

实验结果，我的猴子要经历`876511`次尝试，才能打出单词`cat`。

如果设事件 A = “猴子连续敲击键盘，得到单词cat”，那么：

$$P(A) = {1 \over 85^3} = {1 \over 614125}$$

概率好小！

##实验二：重复实验

于是我开始抽风了= =

我在想，这么小的概率，如果我事先不知道的话，要怎么才能从实验中得到呢？那就多次实验好了。

那么，我要观察的随机变量 X = “猴子从开始直到打出cat为止的尝试次数”，很容易知道，X服从几何分布，即：

$$ P(X = k) = (1 - p)^{k-1}  p $$

其中，k是第一次成功所需的次数， $$$ p = {1 \over 614125}$$$ 。

那么，只要多次实验就好了！

这是这次实验的`Python`脚本：

	import random
	import string
	import csv
	charset = string.ascii_letters + string.punctuation + ' '
	destination = "cat"
	writefile = file('random_letter.csv', 'wb')
	writer = csv.writer(writefile)
	print "All possible Permutations:", len(charset) ** len(destination)
	for time in xrange(10000):
		counter = 1
		while 1:
			generated = "".join(random.choice(charset) for i in xrange(len(destination)))
			if generated == destination:
				break
			else:
				counter += 1
		writer.writerow([counter])

	writefile.close()

嗯，我实验了10000次，花了快两天时间才跑完= =

##线性回归拟合

有了这10000次数据，我们要怎样拟合才好呢？先看看数据的分布如何：

![Distribution-for-monkeys](http://hahastudio.github.com/assets/images/distribution-for-monkeys.png "概率分布")

我将尝试次数按100000长的区间分开，统计每个区间内的出现次数。

从图上很简单就看出，这些柱形的高度满足等比数列，那么，我们就要尝试求一求这个等比数列的方程：

1. 等比数列的通项为 $$$ y = a_1 q^{x-1} $$$，其中 $$$q = (1-p)^{100000}$$$
2. 将上述通项变换 $$$\ln y = \ln {a_1 \over q} + x \ln q $$$
3. 线性拟合，以`Excel`为例，`=LINEST(F1:F19; G1:G19)`，得$$$q = -0.16464273$$$
4. 将q换算为p，以`Excel`为例，`=1/(1-POWER(EXP(H1);1/100000))`，得：
$$ p = {1 \over 607376.2443} $$

##极大似然估计

当时八月份的时候也就到这儿了。

这学期学完概率统计后，我打算用极大似然估计对p进行无偏估计。

对几何分布，其概率密度函数：
$$ f(x) = (1 - p)^{x-1}  p $$
设极大似然函数
$$ \mathcal L ( p )= \prod_{i=1}^n f(x_i| p)$$

则

$$ \mathcal L ( p )= \prod_{i=1}^n (1 - p)^{x_i-1}  p $$

两边求对数：

$$\ln \mathcal L ( p ) = n \ln p + ({\sum_{i=1}^n x_i} -n) \ln (1-p) $$

对$$$p$$$求导：

$$\frac{d \mathcal L ( p )}{d p} = \frac{n}{p} - {\sum_{i=1}^n x_i -n \over 1 -p} $$

令其为0，有：

$$ \frac{n}{p} - {\sum_{i=1}^n x_i -n \over 1 -p} = 0 $$

最终，得p的无偏估计：

$$ \hat{p} = {n \over \sum_{i=1}^n x_i} $$

那么，我们只要求平均数就可以了！

最后， 

$$ \hat{p} = {n \over \sum_{i=1}^n x_i} = \frac{1}{610220.026}$$

##结语

果然知识就是力量= =

有工夫的话把遗传算法的无限猴子写下来= =