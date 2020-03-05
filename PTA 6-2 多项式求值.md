# PTA 6-2 多项式求值

本题要求实现一个函数

<!--more-->
<!-- CreateTime:2018/6/29 15:24:28 -->


<!-- math -->

<!-- 标签： C,算法 -->

本题要求实现一个函数，计算阶数为`n`，系数为`a[0]` `...` `a[n]`的多项式$f(x)=\sum_{i=0}^{n}(a[i]\times x^i)$在`x`点的值。

函数接口定义

```csharp
	double f( int n, double a[], double x );
```

其中`n`是多项式的阶数，`a[]`中存储系数，`x`是给定点。函数须返回多项式`f(x)`的值。

裁判测试程序样例

```c
#include <stdio.h>

#define MAXN 10

double f( int n, double a[], double x );

int main()
{
    int n, i;
    double a[MAXN], x;
	
    scanf("%d %lf", &n, &x);
    for ( i=0; i<=n; i++ )
        scanf(“%lf”, &a[i]);
    printf("%.1f\n", f(n, a, x));
    return 0;
}

/* 你的代码将被嵌在这里 */
```

输入样例

```csharp
2 1.1
1 2.5 -38.7
```

输出样例

```csharp
-43.1
```

可通过代码

```csharp

double f(int n, double a[], double x)
{
	double sum = 0;
	// 如果这时的 n 大于最大的数值，就返回比他小 1 的值
	if (n >= MAXN)
	{
		n = MAXN - 1;
	}

	// 这个值用来做中间的计算，也就是计算 x 的中间计算
	// 为什么 temp 默认值会是 1 ？ 原因就是无论多大的数
	// 100000000^0 等于 1 

	double temp = 1;

	for (int i = 0; i <= n; i++)
	{
		// 第 1 次 是 x^0 刚好就是现在 temp 的值
		sum = sum + a[i] * temp;
		// 进行第 2 次计算 x^1 = x = temp * x
		temp = temp * x;
	}

	return sum;
}


/*
// 里面存在x的多少次方，就需要重新定义一个函数来写，如果直接写在代码，代码很不好看
// 但是因为有时间的限制，所以不能使用这个方式，这个方式是每个 x 都需要重新计算多少次方
// 而比较快的方式是下一次的计算使用上一次计算的结果
// 在工程的开发，要尽量避免这种优化
// 但是在写题目到是可以这样考虑
// 每次计算的 x 的方都比原来的大 1 次，也就是我第 2 次的计算可以用到第 1 次计算的结果
double Pow(double x, int count)
{
	double sum = x;

	// 任何一个数的0次都是等于多少？
	if (0 == count) 
	{
		// 100000000^0
		return 1;
	}

	// 这里使用 i = 1 因为这里的值默认 sum 就是等于 x 
	// 如输入 x^2 那么就是 x = x count = 2
	// 如果这里的 i = 0 开始就会首先设置 sum = x；
	// sum 会循环两次，于是返回 x^3 和需要的不一样
	for (int i = 1; i < count; i++)
	{
		//sum = sum * x;
		sum *= x;
	}

	return sum;
}

double f(int n, double a[], double x)
{
	double sum = 0;
	if (n >= MAXN) 
	{
		n = MAXN - 1;
	}

	for (int i = 0; i <= n; i++)
	{
		sum = sum + a[i] * Pow(x, i);
	}

	return sum;
}


*/
```

考点：

1. 大概的输入

2. 是否可以在下一次运算使用上一次的值

3. 阅读题目能力

第2个考点是有些问题，如果比较会设计的小伙伴，就会写出我注释的代码

在工程使用是建议使用被注释的代码，但是被注释的代码会多了一次循环，于是会运行超时

第3个考点在于一开始的 n 的值，`i <= n`的循环和 `i < n` 的循环次数不相同

另外`for (int i = 0; i < n; i++)`和`for (int i = 1; i < n; i++)`的循环次数也不相同，都是相差 1 ，在于初始化 i 的大小和判断循环。

因为 PTA 没有告诉说代码的输出是什么，而且输出在哪里出错了，所以对于初学者还是比较难的，很多很难知道自己的程序在哪错了。一个建议是使用 CodeBlock 进行调试或者 VisualStudio 调试。

两个调试是不相同，可以看到 CodeBlock 支持比较简单的程序，而且使用也很简单。比较推荐简单的代码使用 CodeBlock ，如果训练的要求是实际使用，那么建议使用 VisualStudio 。可以从安装的时候看到 VisualStuio 很大，而且开始部署环境也是比较困难。但是 VisualStudio 可以开发几乎任何的软件。

下载CodeBlock请到官网：[Download binary](http://www.codeblocks.org/downloads/26 )

下载 VisualStudio 请到官网 [Visual Studio](https://www.visualstudio.com/zh-hans/ )

在部署完成VisualStudio 之后，可以使用我修改的代码运行。需要注意在 VisualStduio 需要使用  `scanf_s` 替换`scanf`，其他几乎不需要修改。

下面的代码复制之后就可以在 VisualStudio 运行调试，注意 VisualStudio

```c
// JisnaicasManawashar.cpp: 定义控制台应用程序的入口点。


#include "stdafx.h"

#define MAXN 10

double f(int n, double a[], double x);

int main()
{
	int n, i;
	double a[MAXN], x;

	//scanf("%d %lf", &n, &x);
	//for (i = 0; i <= n; i++)
	//	scanf("%lf", &a[i]);

	n = 2;
	x = 1.1;
	//1 2.5 -38.7
	a[0] = 1;
	a[1] = 2.5;
	a[2] = -38.7;

	printf("%.1f\n", f(n, a, x));
	return 0;
}


double f(int n, double a[], double x)
{
	double sum = 0;
	// 如果这时的 n 大于最大的数值，就返回比他小 1 的值
	if (n >= MAXN)
	{
		n = MAXN - 1;
	}

	// 这个值用来做中间的计算，也就是计算 x 的中间计算
	// 为什么 temp 默认值会是 1 ？ 原因就是无论多大的数
	// 100000000^0 等于 1 

	double temp = 1;

	for (int i = 0; i <= n; i++)
	{
		// 第 1 次 是 x^0 刚好就是现在 temp 的值
		sum = sum + a[i] * temp;
		// 进行第 2 次计算 x^1 = x = temp * x
		temp = temp * x;

		// 第1次 sum = 1
		// 第2次 sum = 3.75
		// 第3次 sum = -43.1
	}

	return sum;
}


/*
// 里面存在x的多少次方，就需要重新定义一个函数来写，如果直接写在代码，代码很不好看
// 但是因为有时间的限制，所以不能使用这个方式，这个方式是每个 x 都需要重新计算多少次方
// 而比较快的方式是下一次的计算使用上一次计算的结果
// 在工程的开发，要尽量避免这种优化
// 但是在写题目到是可以这样考虑
// 每次计算的 x 的方都比原来的大 1 次，也就是我第 2 次的计算可以用到第 1 次计算的结果
double Pow(double x, int count)
{
	double sum = x;

	// 任何一个数的0次都是等于多少？
	if (0 == count) 
	{
		// 100000000^0
		return 1;
	}

	// 这里使用 i = 1 因为这里的值默认 sum 就是等于 x 
	// 如输入 x^2 那么就是 x = x count = 2
	// 如果这里的 i = 0 开始就会首先设置 sum = x；
	// sum 会循环两次，于是返回 x^3 和需要的不一样
	for (int i = 1; i < count; i++)
	{
		//sum = sum * x;
		sum *= x;
	}

	return sum;
}

double f(int n, double a[], double x)
{
	double sum = 0;
	if (n >= MAXN) 
	{
		n = MAXN - 1;
	}

	for (int i = 0; i <= n; i++)
	{
		sum = sum + a[i] * Pow(x, i);
	}

	return sum;
}


*/
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
