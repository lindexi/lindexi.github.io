# C# 快速释放内存的大数组

本文告诉大家如何使用 Marshal 做出可以快速释放内存的大数组。

最近在做 3D ，需要不断申请一段大内存数组，然后就释放他，但是 C# 对于大内存不是立刻释放，所以就存在一定的性能问题。
在博客园看到了一位大神使用 Marshal 做出快速申请的大数组，于是我就学他的方法来弄一个。本文告诉大家这个类是如何使用。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->


在使用的时候，先来看下原来的 C# 的大数组性能。可以看到在不停gc，性能不好

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201712151723520171220112957.jpg)

```csharp
      static void Main(string[] args)
        {
            for (int i = 0; i < 10000; i++)
            {
                Foo();
            }
            Console.ReadKey();
        }

        private static void Foo()
        {
            var foo = new byte[1000000000];
        }
```

## 介绍

在使用 Marshal 之前需要知道这是什么，其实 Marshal 就是一个提供 COM 互操作的方法。

## 使用

下面使用一个快速申请 int 数组来告诉大家如何使用。

是否还记得 C 的申请数组？其实下面的方法和 C 的相同

```csharp
            int n = 100000;//长度
            IntPtr buffer = Marshal.AllocHGlobal(sizeof(int) * n);
```

这时就可以使用 buffer 作为数组

下面对他的第 k 个元素修改

```csharp
            IntPtr buffer = Marshal.AllocHGlobal(sizeof(int) * n);
            int k = 2;

            IntPtr t = buffer + k * sizeof(int);
            var p = Marshal.PtrToStructure<int>(t);
            Console.WriteLine("p " + p); //196713 这时的值是不确定

            p = 2;
            Marshal.StructureToPtr(p,t,false);

            p = Marshal.PtrToStructure<int>(t);
            Console.WriteLine("p " + p);//2

            //遍历
            Console.WriteLine("遍历");
            for (int i = 0; i < 10; i++)
            {
                t = buffer + i * sizeof(int);
                Console.WriteLine(Marshal.PtrToStructure<int>(t));
            }
```

遍历：

```csharp
            43909312
            44502144
            2
            0
            0
            24
            1357220181
            196712
            550912
            543686656
```

可以从上面的代码看到，主要使用的两个方法是 StructureToPtr 和 PtrToStructure ，而 StructureToPtr 就是从指定类型写到指针，希望大家还知道如何使用指针，PtrToStructure 就是从指针指向的地方开始读数据，读指定类型的数据。所以可以从 Marshal 把一个类型使用另一个类型来读取，但是一般需要读取的类型都需要是确定类型大小的，如 char 可以、string 不可以。

反过来，StructureToPtr 是对指定指针写入指定的类型，同样也是需要确定这个类型的大小，如可以写入 char 但是不可以写入 string。这就是对数组读写的方法。

那么遍历的时候什么输出一些诡异的值，实际上因为没有初始化，里面的值是不确定的。我觉得用这个做随机数也不错。

使用 Marshal 是比较安全，因为 ms 做了很多处理，但是也会让程序闪退，如下面的代码

```csharp
        private static void Foo()
        {
            int n = 100000;//长度
            IntPtr buffer = Marshal.AllocHGlobal(sizeof(int) * n);

            try
            {
                var t = buffer + (n * 10) * sizeof(int);
                var p = Marshal.PtrToStructure<int>(t);
            }
            catch (Exception e)
            {
                Console.WriteLine(e);
            }

            Marshal.FreeHGlobal(buffer);
        }
```

会出现异常 System.AccessViolationException，这个异常是无法 catch 的，所以使用的时候最好封装一下

```csharp
“System.AccessViolationException”类型的未经处理的异常在 未知模块
尝试读取或写入受保护的内存。这通常指示其他内存已损坏
```

如果需要 catch 那么请在 app.config 添加下面的代码

```csharp
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <runtime>
    <legacyCorruptedStateExceptionsPolicy enabled="true" />
  </runtime>
</configuration>
```

然后在 Main 函数添加 HandleProcessCorruptedStateExceptions ，请看代码

```csharp
        [HandleProcessCorruptedStateExceptions]
        static void Main(string[] args)
        {
            AppDomain.CurrentDomain.UnhandledException += CurrentDomain_UnhandledException;

            for (int i = 0; i < 100000; i++)
            {
                try
                {
                    Foo();
                }
                catch (Exception e)
                {
                    Console.WriteLine(e);
                   
                }
            }
            Console.WriteLine("完成");
            Console.ReadKey();
        }
```

这时可以看到进入 UnhandledException ，但是无法接住，软件还是会崩溃

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201712151723520171220152536.jpg)

### 释放内存

那么如何释放内存？因为这个申请是没有经过管理的，如果没有手动释放，那么就出现内存泄露。

```csharp
        static void Main(string[] args)
        {
            for (int i = 0; i < 10000; i++)
            {
                Foo();
            }
            Console.ReadKey();
        }

        private static void Foo()
        {
            int n = 100000;//长度
            IntPtr buffer = Marshal.AllocHGlobal(sizeof(int) * n);
           
        }
```

上面的代码很快就可以看到内存占用到2G，所以需要手动释放

```csharp
            Marshal.FreeHGlobal(buffer);

```

原来的 byte 数组需要使用 1G 内存，而且速度很慢，而现在使用这个方法只需要 7M 内存，速度很快

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201712151723520171220142918.jpg)

所以在需要进行大数组申请的时候，需要不停释放，就可以使用这个方法。

如果想使用封装好的，请看下面的大神弄好的类

参见：[C#+无unsafe的非托管大数组(large unmanaged array in c# without 'unsafe' keyword) - BIT祝威 - 博客园](http://www.cnblogs.com/bitzhuwei/p/huge-unmanged-array-in-csharp.html )

## 实际使用

实际在哪些地方使用？实际上因为很多时候都是使用实例化池，但是实例化池在进入游戏的时候，可以让gc不会让程序暂停，但是会在游戏进入下一关的时候，无法快速清理数据。所以这时就可以使用 Marshal 做实例化池，瞬间就可以清空。

上面的方法暂时不告诉大家如何做，因为涉及到公司的使用。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  