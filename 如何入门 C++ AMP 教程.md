# 如何入门 C++ AMP 教程

本文告诉大家如何写一个 Helloworld 程序。

<!--more-->
<!-- CreateTime:2019/11/29 8:20:37 -->


首先打开 VisualStudio ，大概现在也没有人还在用 VisualStudio 2013 了，所以我就不需要告诉大家需要用哪个版本的 VisualStudio ，如果我发现你的 VisualStudio 版本不对让程序运行错误，我就不会去回复。

创建一个空白的 C++ 项目，请看图片

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017123143444.jpg)

然后就可以看到这个项目没有什么东西，接着右击源文件，添加C++文件

然后输入下面代码

```c
#include <iostream> 
#include <amp.h> 
using namespace concurrency;
int main()
{
	int v[11] = { 'G', 'd', 'k', 'k', 'n', 31, 'v', 'n', 'q', 'k', 'c' };

	array_view<int> av(11, v);
	parallel_for_each(av.extent, [=](index<1> idx) restrict(amp)
	{
		av[idx] += 1;
	});

	for (unsigned int i = 0; i < 11; i++)
	{
		std::cout <<static_cast<char>(av[i]);
	}
}
```

这里的 v 就是 Helloworld 减去 1 得到的，所以进行计算输出

按下运行就可以看到输出

这个例子是多线程同时执行，需要进行多线程的计算不能有依赖，也就是有需要计算的函数 1 和 2 不能让 1 需要等待 2 执行完成才能运行，只有 1 和 2 能同时运行的才能使用这个方法

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。