
# dotnet 里的那些锁 AutoResetEvent 用法

本文告诉大家在 dotnet 里的 AutoResetEvent 锁的用法

<!--more-->


<!-- 发布 -->

## 用法

使用 WaitOne 等待，使用 Set 让等待的逻辑继续执行

```csharp
        private static void Foo(AutoResetEvent autoResetEvent)
        {
            Task.Run(() =>
            {
                while (true)
                {
                    autoResetEvent.WaitOne();

                    Console.WriteLine("Foo");
                }
            });
        }

        static void Main(string[] args)
        {
            var autoResetEvent = new AutoResetEvent(false);

            Foo(autoResetEvent);

            autoResetEvent.Set();

            Console.Read();
        }
```

## 作用

多次调用 Set 时，只有同一时间的 WaitOne 执行，如以下代码

```csharp
        static void Main(string[] args)
        {
            var autoResetEvent = new AutoResetEvent(false);

            Foo(autoResetEvent);

            autoResetEvent.Set();

            for (int i = 0; i < 10; i++)
            {
                autoResetEvent.Set();
            }

            Console.Read();
        }
```

上面代码放在[github](https://github.com/lindexi/lindexi_gd/tree/b26e1d5d5fe1cb6be891d4849dd2d15a8681271b/HaifeljiweajeYeelarkerjairere)欢迎小伙伴访问 

在 for 里面做快速的释放，而 Foo 则是本文上面的方法，只会输出一次 `Foo` 执行一次

因此 AutoResetEvent 能做到多次设置执行，最终只有一次执行

**在 WaitOne 执行之前，无论使用 Set 多少次，最终只能执行一次 WaitOne 方法**

因此，如果在 WaitOne 执行之后，再次调用 Set 方法，那么将会继续让其他的 WaitOne 执行。下面代码在 Set 方法之后调用Thread.Sleep方法，让 Foo 方法的线程 WaitOne 执行

```csharp
            for (int i = 0; i < 10; i++)
            {
                autoResetEvent.Set();
                Thread.Sleep(100);
            }
```

上面代码放在[github](https://github.com/lindexi/lindexi_gd/tree/509fb7594f9f1fc1215cd7f4e4127c90354fb9d2/HaifeljiweajeYeelarkerjairere)欢迎小伙伴访问

如果有多个线程同时等待 WaitOne 方法，那么调用 Set 方法，只有一个线程执行。如果同时多次调用 Set 方法，最终也只有一次 WaitOne 之后的逻辑执行。只有在 WaitOne 通过之后的 Set 方法，才会让下一个 WaitOne 执行

如复制 Foo 方法，更改命名为 Foo2 方法，然后修改输出为 `Foo2` 在主函数执行

```csharp
        static void Main(string[] args)
        {
            var autoResetEvent = new AutoResetEvent(false);

            Foo(autoResetEvent);
            Foo2(autoResetEvent);

            autoResetEvent.Set();

            for (int i = 0; i < 10; i++)
            {
                autoResetEvent.Set();
                Thread.Sleep(100);
            }

            Console.Read();
        }

        private static void Foo2(AutoResetEvent autoResetEvent)
        {
            Task.Run(() =>
            {
                while (true)
                {
                    autoResetEvent.WaitOne();

                    Console.WriteLine("Foo2");
                }
            });
        }
```

上面代码让 Foo 和 Foo2 两个方法进入 WaitOne 方法，而执行输出每次只有一个线程执行，运行代码可以看到输出是

```csharp
Foo2
Foo
Foo2
Foo
Foo2
Foo
Foo2
Foo
Foo2
Foo
```

其中的 Set 调用了 11  次，但是第一次和第二次是合并执行，于是 WaitOne 实际只执行了 10 次。分别给 Foo 和 Foo2 两个方法执行

上面代码放在[github](https://github.com/lindexi/lindexi_gd/tree/e1a0b2870785279cbe81788e6f62892229279103/HaifeljiweajeYeelarkerjairere)欢迎小伙伴访问

## 异步锁

以上 AutoResetEvent 类将会占用线程，没有提供异步的方法。如果想要 AsyncAutoResetEvent 请使用 [dotnet-campus/AsyncWorkerCollection: 多线程异步工具](https://github.com/dotnet-campus/AsyncWorkerCollection ) 开源仓库的方法

通过 NuGet 安装 dotnetCampus.AsyncWorkerCollection 库，如果是 SDK 风格的 csproj 文件，可以添加如下代码

```xml
<PackageReference Include="dotnetCampus.AsyncWorkerCollection" Version="1.1.6" />
```

当然， 不想添加多一个 dll 的小伙伴，可以采用源代码版本，以下 NuGet 包是通过 [SourceYard](https://github.com/dotnet-campus/SourceYard) 创建的源代码 NuGet 包，安装 NuGet 包将相当于将代码拷贝到项目

```
dotnet add package dotnetCampus.AsyncWorkerCollection.Source --version 1.1.6
```

如果是 SDK 风格的 csproj 文件，可以添加如下代码

```xml
<PackageReference Include="dotnetCampus.AsyncWorkerCollection.Source" Version="1.1.6">
  <PrivateAssets>all</PrivateAssets>
  <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
</PackageReference>
```

使用 AsyncAutoResetEvent 可以将 WaitOne 替换为 WaitOneAsync 方法





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。