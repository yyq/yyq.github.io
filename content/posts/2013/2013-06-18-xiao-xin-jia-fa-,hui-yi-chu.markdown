---
layout: post
title: "小心加法的溢出"
date: 2013-06-18 00:06:01
comments: true
---
过几天又有一次微软的笔试了，还是把手头上ios的进度缓一缓。随意翻翻编程之美吧。今天看了一到题目，如何写出正确的二分查找。想起来，看上去不会很难。但是一上来就错了。。。。。

1. mid = (smalll + big) /2; 这个句子中的加法，是有可能溢出造成结果不对。所以推荐用

	mid = small + ( big - small ) / 2;
	
2. 对for循环或者while 或者递归等，每当自己写完之后，观察边界：初始条件，转化，终止条件

3. 突然想到的代码，到底能不能正常运行：
		{% codeblock %}
		#include <stdio.h>
		int main(int argc, char *argv[]) {
		int a[3] = {1,2,3};
	
		int *b = a;
		int *c = &a[2];
	
		int *d;
	
		d = (int *)((int)b / 2 + (int)c / 2);
		
		printf("%d %d %d\n", *b , *c, d);
	
		return 0;
		}
		{% endcodeblock %}