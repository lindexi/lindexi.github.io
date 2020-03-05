# dotnet 线程静态字段

在 dotnet 程序提供了一个好用的特性，可以让字段作为线程的静态字段，也就是在相同线程的所有代码访问的静态字段是相同对象，但不同线程访问的时候是不同的

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在 .NET 程序可以使用 ThreadStaticAttribute 特性标记在一个静态字段上，这样这个字段就可以做到在线程里面静态

在一个类的静态字段上面添加 [ThreadStaticAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.threadstaticattribute?wt.mc_id=MVP ) 可以让这个字段作为线程的静态字段，也就是在相同的线程访问的时候这个字段是静态的，拿到的对象的实例相同，但是在不同的线程拿到不相同

在 [ThreadStaticAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.threadstaticattribute?wt.mc_id=MVP ) 支持的只有静态字段，不支持静态属性，不支持普通的字段。同时添加了这个特性的静态字段不支持初始化

下面写一段代码尝试一下

```csharp
    class Foo
    {
        public static string StaticProperty
        {
            get => _staticProperty;
            set => _staticProperty = value;
        }

        public static string ThreadStaticProperty
        {
            get => _threadStaticProperty;
            set => _threadStaticProperty = value;
        }

        [ThreadStatic] private static string _threadStaticProperty = "初始值";
        private static string _staticProperty = "初始值";
    }
```

我在一个类创建了两个不同的静态属性，一个是普通的静态属性，另一个是线程静态属性，我尝试都给两个字段初始值

```csharp
        static void Main(string[] args)
        {
            Foo.StaticProperty = "普通静态属性";
            Foo.ThreadStaticProperty = "线程静态属性";

            var taskList = new List<Task>();

            for (int i = 0; i < 10; i++)
            {
                var n = i;
                var task = new Task(() =>
                {
                    Console.WriteLine(
                        $"thread={Thread.CurrentThread.ManagedThreadId} 静态属性={Foo.StaticProperty} 线程静态属性={Foo.ThreadStaticProperty} 次数={n}");

                    Foo.StaticProperty = n.ToString();
                    Foo.ThreadStaticProperty = n.ToString();
                });

                task.Start();
                taskList.Add(task);
            }

            Task.WaitAll(taskList.ToArray());
        }

```

我添加了上面的代码用于多个线程输出值同时修改值，在运行的时候可以看到，对于线程静态属性的输出都是空，即使我在代码添加了初始值。因为线程静态属性不支持给初始值，另外在不同的线程修改的线程静态属性是不会影响其他线程

上面代码的输出如下，可能小伙伴运行的输出和我不相同

```csharp
thread=9 静态属性=普通静态属性 线程静态属性= 次数=4
thread=10 静态属性=普通静态属性 线程静态属性= 次数=3
thread=4 静态属性=普通静态属性 线程静态属性= 次数=1
thread=11 静态属性=普通静态属性 线程静态属性= 次数=7
thread=5 静态属性=普通静态属性 线程静态属性= 次数=0
thread=6 静态属性=普通静态属性 线程静态属性= 次数=2
thread=7 静态属性=普通静态属性 线程静态属性= 次数=5
thread=8 静态属性=普通静态属性 线程静态属性= 次数=6
thread=10 静态属性=3 线程静态属性=3 次数=9
thread=9 静态属性=3 线程静态属性=4 次数=8
```

从上面代码可以知道如果想要多个线程之间的静态字段或属性不相互影响，可以通过 [ThreadStaticAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.threadstaticattribute?wt.mc_id=MVP ) 如输出的最后两行，可以看到普通静态属性是在所有线程使用相同实例，于是输出的静态属性的值相同。但是线程静态属性是每个线程不相同的，在线程 10 的次数是 3 修改的属性值也就是 3 最后输出的就是 3 同时在线程 9 里面的线程静态属性和上次线程修改的相同

本文用到的类放在[github](https://github.com/lindexi/lindexi_gd/tree/4038b7f55a836a491291e0b50abdbc21b10ce093/NiwewheejaiKerebawkaykerego ) 欢迎小伙伴访问

[ThreadStaticAttribute](https://docs.microsoft.com/en-us/dotnet/api/system.threadstaticattribute?wt.mc_id=MVP )

[ThreadStatic静态字段在每个线程里的唯一性 - 王树伦的博客 - CSDN博客](https://blog.csdn.net/wshl1234567/article/details/50820503 )

[C# [ThreadStatic] 标记静态字段对多线程执行的影响 - Ryan_zheng - 博客园](https://www.cnblogs.com/ryanzheng/p/10962513.html )

[ThreadStatic特性标记静态字段对多线程执行的影响 - b0b0 - 博客园](https://www.cnblogs.com/hbb0b0/archive/2011/01/14/1935587.html )

[在多线程中使用静态方法是否有线程安全问题 - 逍遥剑客的专栏 - CSDN博客](https://blog.csdn.net/scucj/article/details/1394523 )

[C#静态变量和静态方法的线程安全问题 - littwo - CSDN博客](https://blog.csdn.net/littwo78168218/article/details/17526471 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
