---
title: 图解马拉车算法及其Golang实现
poster:
  topic: 标题上方的小字
  headline: 大标题
  caption: 标题下方的小字
  color: 标题颜色
date: 2023-02-01 18:30:45
url: illustratbe-manacher-algorithm-zh
cover:
banner: algorithm
h1:
categories:
tags:
mathjax: true
---



最近发现网上普遍对马拉车算法（Manacher's Algorithm）的讲解有些拘泥于算法细节。这片文章只介绍原理，实现的细节可直接参考最后的代码实现。

<!--more-->

## 什么是马拉车算法

马拉车算法能以 $O(n)$ 的时间复杂度求得字符串$s$的最长子回文。

如果读不懂上面👆🏻这句话，可以参考这道题目👉🏻 [Leetcode 5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)。

最长回文子串问题有一个朴素的 $O(n^2)$ 的算法很容易想到，就是枚举对称中心，以对称中心为起点向$s$两侧扩散，这叫做中心扩散法。

实际上，**马拉车算法就中心扩散法的优化**。

## 怎么优化

假设 $s=cbcbccde$, 我们开一个数组 $p$ ， $p[i]$ 表示以 $s[i]$ 为对称中心的那个最长的回文子串的半径长度。这样说可能比较拗口，举个例子就明白了，如图1。

{% image 1.png 图1. 例，因为以s[2]为中心的最长回文是cbcbc，半径为3，故p[2]=3 。  width:400px padding:20px bg:white  fancybox:true download:true %}

 $p$ 数组的值是很容易利用中心扩散法求解出来的，只是这样做时间复杂度是$O(n^2)$的。马拉车算法就是利用 $p$ 中已知的信息将算法优化到了$O(n)$。下面举个例子，假设目前 $p[0], p[1], p[2]$ 已知，现在我们要求出$p[3]$ ，如图2所示。

{% image 2.png 图2. p[0~2]已知，欲利用已知信息求得p[3] 。  width:400px padding:20px bg:white  fancybox:true download:true %}

下面阐述一个事实：已知的右界最靠右的最长子回文的对称中心是$s[2]$, 其右界是$s[4]$。这个事实看图很好理解，如图3所示。

{% image 3.png 图3. 红线是对称轴，s[4]是当前已知的最右右界 。  width:400px padding:20px bg:white  fancybox:true download:true %}

承认了上面的事实就好办了，此时，我们要求的p[3]位于下标3，处于中心2和右界4的区间内。根据对称性，$p[3]\ge p[2-(3-2)] = 2$ 马拉车算法的精髓就在这一步。 其中 $p[3]\ge p[2-(3-2)]$ 就是 $p[1]$ , 是 $p[3]$ 关于图3中红色对称轴的对称点。

## 一些特殊情况

上面为什么说$p[3] \ge p[1]$ ，而不是$p[3] = p[1]$呢，我们看下面图4中的例子，对于这个例子，很显然$p[3]$的真实值是3，比$p[1]$的要大。

这个原因很简单，就是因为我们已知的信息是有限的，即使我们利用了已知的最右的右界，真实的情况却是$p[3]$的右界还是超出了最右右界的范围。

为了处理这种情况，我们通过马拉车算法算出一个$p[i]$后（ 此时这个$p[i]$可能小于真实值），只需以$s[i]$为对称中心，以半径$p[i]$为起始值执行中心扩散，并不断更新$p[i]$。在图4的例子中，$i=3$。

{% image 4.png 图4. 注意这里为了说明问题，对样例稍加改动，s[5]=b 。  width:400px padding:20px bg:white  fancybox:true download:true %}



还有一种特殊情况，就是利用已知信息计算 $p[i]$ 只有在 $i$ 处于图3中的红色对称轴和对应的右界之间的时候才可行，其它情况直接采用中心扩散法计算 $p[i]$ 即可。



## 关于偶回文

上面的例子我们为了直达核心，都只考虑了奇回文（回文长度为奇数），忽略了偶回文的求解。偶回文的麻烦之处在于它的对称轴在两个字符之间。我们可以通过一个简单的小trick来把偶回文的问题转化为奇回文。就是把字符串$s$中的字符用一个符号隔开，比如 $abac$ 变成 $$ \# a \# b \# a \# c \#$$ 。这样操作后，偶回文就转化成了奇回文，奇回文仍是奇回文。



## Golang 实现

下面是 [Leetcode 5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/) 的代码。

```go
func Min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

func longestPalindrome(s string) string {
	ss := make([]byte, len(s)*2+1)
	ss[0] = '#'
	si := 1
	for i := 0; i < len(s); i++ {
		ss[si] = s[i]
		si++
		ss[si] = '#'
		si++
	}

	// Manacher's algorithm
	p := make([]int, len(ss))
	mid, maxRight := 0, 0
	maxPos := 0
	for i := 0; i < len(ss); i++ {
		if i < maxRight {
			p[i] = Min(p[2*mid-i], maxRight-i)
		} else {
			p[i] = 1
		}
		for i+p[i] < len(ss) && i-p[i] > -1 && ss[i+p[i]] == ss[i-p[i]] {
			p[i]++
		}
		if i+p[i] > maxRight {
			mid = i
			maxRight = i + p[i]
		}
	}
	for i := 0; i < len(p); i++ {
		if p[i] > p[maxPos] {
			maxPos = i
		}
	}
	return s[(maxPos-p[maxPos]+1)/2 : (maxPos+p[maxPos]-1)/2]
}
```



测评结果：



{% image 5.png  width:400px padding:20px bg:white  fancybox:true download:true %}



