---
layout: post
title: "C Traps and Pitfalls"
date: 2013-05-29 07:49:01
comments: true
tags: [c]
---
了解到一本很经典的C语言的书，我没有读过，《C陷阱与缺陷》，此书不厚，但是经典。这片博客用来做读书笔记吧。

1. printf("%d",010);
	
	输出是8。平常一不小心就错了。
	
2. 词法分析中的贪心法：a---b的意思是 (a--)-b,而不是a-(--b);因为编译器从左到右扫描，最长的一个有意义的字符是 --；
3. **a=b/*p** 肯定会报错，因为 **/*** 会被解析成注释的前缀。
	
	正确的表达方式应该是**a= b / *p** 或者 **a = b /(*p)**
4. **printf('\n');** 是错误的；**printf("\n")** 是正确的；

	char Hello[]={'h','e','l','l','o','w','o','r','l','d','\n','0'};
	printf(Hello);
	<!--more-->
5. void
	```
	(*(void(*)()0)) (); 
	```
	独立运行于某种微处理器上的C程序，当计算机启动时，硬件将调用首地址为0位置的子进程。为了模拟开机启动时的清醒，设计一个C语句，显式调用孩子进程。
	
	* float *g():代表一个函数，返回值为float型指针
	* floag (*h)():代表一个函数指针，这个指针指向一个返回float变量的函数。
	* 假设fp是一个函数指针，那么调用那个函数就是{% codeblock %}(\*fp)();{% endcodeblock %},{% codeblock %}(\*0)(){% endcodeblock %}则表示调用位置在0的函数了。
	* 通过如下可以完成调用存储位置为0的子进程	
	{% codeblock %}
	void (*fp)();
	or
	(*fp)();
	{% endcodeblock %}
	
	* 一旦知道声明一个变量，就知道如何对一个常数进行类型转换，将其转型为该变量的类型，只需要在变量声明中，把变量名去掉即可，将常数0转换为“指向返回值为float的函数的指针”类型，可以写成：
	
	```
	( float(*)() )0
	```

6. 运算符优先级，单目运算 > 双目运算 > 三目运算

	双目运算符中:算术运算符 > 移位运算符 > 关系运算符 > 逻辑运算符 > 赋值运算符 > 条件运算符(三目运算符)

	运算所需变量为一个的运算符叫单目运算符
	
	* 任何一个逻辑运算符的优先级都低于任何一个关系运算符
	* 移位运算符比算术运算符要低

7. 错误表达方式：
	
		while (c=getc(in) != EOF)
			putc(c,out);

   正确表达方式：

		while ( (c=getc(in)) != EOF) 
			 putc(c,out);
			 
8. extern
 	
 	在所有的函数体之外： 
 	 
 	 {% codeblock %}
 	 int a = 7; //是一个声明语句，在定义a的同时，也为a明确指定了初始值。
 	 extern int a; //从链接器的角度来看，这个声明是对外部变量a的引用，不是对a的定义。
 	 {% endcodeblock %}
 
  	如果在文件1.c中，声明了int a = 7,文件2.c中声明了extern int a;那么，就不可以有其他的文件中再出现int a = …因为如果这样，显然链接器会不知所措。分别编译文件时可以OK,但是在link的时候就会报错了。
  	
  	这个时候，**static**则可以起到作用了。static int a;这样一个句子，就会**限定了变量a的作用域在一个源文件之内**，对于其他源文件a是不可见的。
  
9. 命名冲突和static
		
	如果一个库函数需要调用另一个未在ANSI C标准中列出的库函数，那么它应该以“隐藏名称”来调用后者。这就使得程序员可以定义一个函数，比如函数名为read，而不用担心库函数getc本应调用库文件中的read函数，去调用了这个用户定义的read函数，但大多数C语言实现并不是这样做的，因此这类命名冲突仍然是一个问题。
	
	static修饰符是一个能够减少此类命名冲突的有用工具。static修饰符不近适用于变量，也适用于函数。如果函数f需要调用另一个函数g，而且只有函数f需要调用函数g，我们可以吧函数f与函数g都放到同一个源文件中，并且声明g为static；

10. 如果一个函数在被定义或者声明之前调用，那么它的返回类型就默认为整形。

	文件1.c   
		{% codeblock %}
		int square(int a);
		main(){ printf("%d\n",square(3));}
		{% endcodeblock %}
	文件2.c
		{% codeblock %}
		int square(int a){ return a*a; }
		{% endcodeblock %}
	
	正常情况下，返回为9。若把文件1中的声明去掉，返回值也为9.
	若改成如下这个样子：
		{% codeblock %}
		main(){ printf("%lf\n",square(3));}
		{% endcodeblock %}
	文件3.c
		{% codeblock %}
		double square(int a){ return a*a*1.0; }
		{% endcodeblock %}
  	那文件1的输出就是0.000000了
  	
  	如果square的调用和定义分别位于不同的文件中，那么我们必须在调用它的文件中声明square函数。
  
11. 宏
	
	宏定义中需要注意**空格**和**括号**
	
	**example1:**
   
   	{% codeblock %}
   	#define abs(x) x>0?x:-x
   	{% endcodeblock %} 
   	这个定义是错误的,
   
   	因为{% codeblock %}abs(a-b){% endcodeblock %}这个算式宏展开之后，
   	
   	就是{% codeblock %}a-b>0?a-b:-a-b{% endcodeblock %},而这显然不是我们想要的结果。
   	
   	而{% codeblock %}#define abs(x) (((x)>=0)?(x):-(x)){% endcodeblock %}看上去更像是正确答案
   	
   	**example2:**
   	
   	看看这个{% codeblock %}#define max(a,b) ((a)>(b)?(a):(b)){% endcodeblock %},好像也是正确的样子.
   	
   	但是仍然存在其他的问题。**一个操作数如果在两处被用到，就会被求值两次。**这个算式中，如果实际情况是a>b的话，那么a会被求值两次：第一次在a和b比较的时候，第二次在max求结果的时候。下面来看看可能的错误情形:
   			
   		{% codeblock %}
   		biggest = x[10];
   		i = 1;
   		while( i < n )
   			biggest = max(biggest , x[i++]); 		{% endcodeblock %}
   	
   	这个过程就是要找出数组中最大的数。就是这里了，注意到max宏调用时，给进去的第二个数据中的i++;宏展开后看一下：
   	
   	{% codeblock %}
   	(biggest)>(x[i++])?(biggest):(x[i++])
   	{% endcodeblock %}
   	
   	在比较完成时，i已经++了，然后给max求值时，若是选择冒号后面的算式的话，后果不堪设想了。
   	
   	**example3**
   	
   	宏的另外一个危险就是：宏展开可能产生非常庞大的表达式，占用的空间远远超过了我们所期望的空间。比如上述的那个max宏。如果我们需要找到abcd四个数中的最大值，利用上面的宏来写，最方便的写法就是{% codeblock %}max(a,max(b,(max(c,d)))){% endcodeblock %}，展开后就是。。。。太长了，我不想写。若是把刚才的写法稍微改一下{% codeblock %}max(max(a,b),max(c,d)){% endcodeblock %}则会好一些，但是还是略长。
   	
   	其实，写成如下的代码似乎程序运行效率更高一些：

   		biggest = a;
   		if(b>biggest) b = biggest;
   		if(c>biggest) c = biggest;
   		if(d>biggest) d = biggest;
   	
   	**example4**
   	
   	宏不适合定义类型
   	
   		#define T1 struct foo *
   		typedef struct foo *T2
   		T1 a,b;
   		T2 c,d;
   	
   	上面句子中的第三行就会出现问题了，struct foo *a, b;那么a和b的类型就一个是指针，一个是结构了。
   	
12. 可移植性的一个例子

	下面的程序接受两个参数：一个long型整数和一个函数指针。这段程序的作用是把给出的long型整数转换为其10进制表示，并且对10进制表示中的每个字符都调用函数指针所指向的函数。
		{% codeblock %}
		void printnum( long n, void(*p)()){
			if( n < 0 ){
					(*p)('-');
					n = -n;
			}
			if( n >= 10 )
					printnum(n/10 , p);
			(*p) ((int)(n%10) + '0');
		}	
		{% endcodeblock %}

	**问题1**

	在最后一行传入数字字符的时候，使用的是数字加上‘0’，这样的写法，在ASCII字符集是有效的，换了其他的就不一定了。可以考虑用另外一种更为有效的表达方法，用一张代表数字的字符表。因为一个字符串常量可以用来表示一个字符数组，所以在数组名出现的地方都可以用字符串常量来替换。如下：

	{% codeblock %}"0123456789"[ n % 10]{% endcodeblock %}

	**问题2**
	
	在n=-n的这个操作中，很容易就考虑到，如果是基于2的补码的计算机，很有可能溢出了。比如-32768的相反数肯定就不是32768。因此，如果我们保证不将n转换为对应的正数，则可以避免这一个问题。如果n是正数，则转换成负数来处理即可。在此之后的所有算术运算针对负数进行。修改后代码如下：
		{% codeblock %}
		void printneg(long n, void(*p)()){
			if( n <= 10){
				printneg(n/10 , p);
			}
			(*p)("0123456789"[-(n%10)]);
		}
		void printnum(long n, void(*p)()){
			if( n < 0){
				(*p)('-');
			printneg(n,p);
			}else
				printneg(-n,p);
		}	
		{% endcodeblock %}
		
	**问题3**

	当n为负数的时候，n%10也有可能为正数，那-n%10就有可能为负数，那么"0123456789"[-(n%10)]就不存在了。那我们创建两个临时变量来分别保存商和余数。在除法完成之后，看余数是否在合理范围（即正数从0到9）；如果不是，则调整。printnum不用改，改改printneg.
		{% codeblock %}
		void printneg( long n , void (*p)()){
			long q;
			int r;
			q = n / 10;
			r = n % 10;
			if( r> 0){
				r -= 10; q ++;
			}
			if( n <= -10)
				printneg( q , p);
			(*p)("0123456789"[-r]);
		}	
		{% endcodeblock %}

## 小节
至此，c陷阱与缺陷这本书算是看完第一遍了。感觉自己对c又有了新的认识。用书中译者的话来说就是，在C语言最为幽微晦暗的部分探奇揽胜。能从书中感受到，这些问题来自于作者多年的实际经验，才会有如此多的case。很多问题，作者也没有去深究为何，但是都给出了解决问题的方法或者思路，很多的陷阱都有很多的方式可以绕过去。就看平时自己的积累了。 	