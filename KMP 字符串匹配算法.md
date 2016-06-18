---
title: KMP 字符串匹配算法
tags: KMP,字符串匹配
grammar_cjkRuby: true
---

[TOC]

[从头到尾彻底理解KMP][1]

**问题描述：**
> 在文本串 `S` 中查找模式 `P`。

##  暴力匹配算法
如果用暴力匹配的思路，并假设现在文本串 `S` 匹配到 `i` 位置，模式串 `P` 匹配到 `j` 位置，则有：
* 如果当前字符匹配成功（即 `S[i] == P[j]`），则 `i++`，`j++`，继续匹配下一个字符；
* 如果失配（即 `S[i]! = P[j]`），令 `i = i - (j - 1)`，`j = 0`。相当于每次匹配失败时，`i` 回溯，`j` 被置为0。
```cpp?linenums
int ViolentMatch(char* s, char* p)
{
	int sLen = strlen(s);
	int pLen = strlen(p);

	int i = 0;
	int j = 0;
	while (i < sLen && j < pLen)
	{
		if (s[i] == p[j])
		{
			//①如果当前字符匹配成功（即S[i] == P[j]），则i++，j++    
			i++;
			j++;
		}
		else
		{
			//②如果失配（即S[i]! = P[j]），令i = i - (j - 1)，j = 0    
			i = i - j + 1;
			j = 0;
		}
	}
	//匹配成功，返回模式串p在文本串s中的位置，否则返回-1
	if (j == pLen)
		return i - j;
	else
		return -1;
}
```
##  KMP 算法
算法流程：
* 如果`j = -1`，或者当前字符匹配成功（即 `S[i] == P[j]`），都令 `i++`，`j++`，继续匹配下一个字符；
*如果 `j != -1`，且当前字符匹配失败（即 `S[i] != P[j]`），则令 `i` 不变，`j = next[j]`。此举意味着失配时，模式串 `P` 相对于文本串 `S` 向右移动了 `j - next [j]` 位
* 换言之，当匹配失败时，模式串向右移动的位数为：失配字符所在位置 - 失配字符对应的next 值，即**移动的实际位数为：j - next[j]**，且此值大于等于1。

```cpp?linenums
int KmpSearch(char* s, char* p)
{
	int i = 0;
	int j = 0;
	int sLen = strlen(s);
	int pLen = strlen(p);
	while (i < sLen && j < pLen)
	{
		//①如果j = -1，或者当前字符匹配成功（即S[i] == P[j]），都令i++，j++    
		if (j == -1 || s[i] == p[j])
		{
			i++;
			j++;
		}
		else
		{
			//②如果j != -1，且当前字符匹配失败（即S[i] != P[j]），则令 i 不变，j = next[j]    
			//next[j]即为j所对应的next值      
			j = next[j];
		}
	}
	if (j == pLen)
		return i - j;
	else
		return -1;
}
```

```cpp?linenums
void GetNext(char* p,int next[])
{
	int pLen = strlen(p);
	next[0] = -1;
	int k = -1;
	int j = 0;
	while (j < pLen - 1)
	{
		//p[k]表示前缀，p[j]表示后缀
		if (k == -1 || p[j] == p[k]) 
		{
			++k;
			++j;
			next[j] = k;
		}
		else 
		{
			k = next[k];
		}
	}
}
```

```cpp?linenums
//优化过后的next 数组求法
void GetNextval(char* p, int next[])
{
	int pLen = strlen(p);
	next[0] = -1;
	int k = -1;
	int j = 0;
	while (j < pLen - 1)
	{
		//p[k]表示前缀，p[j]表示后缀  
		if (k == -1 || p[j] == p[k])
		{
			++j;
			++k;
			//较之前next数组求法，改动在下面4行
			if (p[j] != p[k])
				next[j] = k;   //之前只有这一行
			else
				//因为不能出现p[j] = p[ next[j ]]，所以当出现时需要继续递归，k = next[k] = next[next[k]]
				next[j] = next[k];
		}
		else
		{
			k = next[k];
		}
	}
}

```






























  [1]: http://blog.csdn.net/v_july_v/article/details/7041827