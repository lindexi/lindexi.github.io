# C＃ 局部函数与事件

本文告诉大家使用局部函数可能遇到的坑。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


在以前，如果有一个事件`public event EventHandler Foo`和一个函数`private void Program_Foo(object sender, EventArgs e)`那么使用函数监听事件是很简单的，当然从事件取消函数也是很简单。请看下面代码

```csharp
            for (int i = 0; i < 100; i++)
            {
                Foo -= Program_Foo;
                Foo += Program_Foo;
            }
            Console.WriteLine(Foo.GetInvocationList().Length);
```

结果输出 1

因为`GetInvocationList`是获得事件有多少监听，从上面代码看到，只有一个监听。

如果把函数修改为局部，请看代码

```csharp
            for (int i = 0; i < 100; i++)
            {
                Foo -= Program_Foo;
                Foo += Program_Foo;
            }
            Console.WriteLine(Foo.GetInvocationList().Length);

            void Program_Foo(object sender, EventArgs e)
            {
                
            }
```

现在他会输出什么？

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171024152546.jpg)

看起来没有问题，但是如果再做出一些修改，请看下面代码

```csharp
            for (int i = 0; i < 100; i++)
            {
                F();
            }
            Console.WriteLine(Foo.GetInvocationList().Length);

        private static void F()
        {
            Foo -= Program_Foo;
            Foo += Program_Foo;

            void Program_Foo(object sender, EventArgs e)
            {

            }
        }
```

现在输出是什么？

还是 1

所以可以直接使用局部函数

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。