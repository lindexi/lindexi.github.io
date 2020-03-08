# dotnet 判断特定进程存在方法

本文告诉大家几个方法判断特定的进程是否存在，同时对比这些方法的性能

<!--more-->
<!-- CreateTime:2019/11/29 8:34:18 -->

<!-- csdn -->

## 使用锁判断

在 C# 中判断一个进程存在的方法，如果这个进程是自己创建的，可以通过 Mutex 的方法，通过创建一个锁，然后在其他进程判断这个锁是否存在。这使用到内核的方法，性能不错

假设需要判断进程 HacurbonefeciloQicejewarrerai 是否存在，而这个进程是自己写的进程，那么可以在这个进程的主函数创建一个锁请看代码

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            var mutex = new Mutex(true, Const.Lock, out var createdNew);

            if (!createdNew)
            {
                Console.WriteLine("已经有进程启动");
            }

            Console.ReadKey();

            mutex.Dispose();
        }
    }

    public static class Const
    {
        public const string Lock = "5742D257-CCCC-4F7A-2191-6362609C452D";
    }
```

在另一个进程可以使用下面方法判断进程是否已经存在

```csharp
        public bool FindExistByMutex()
        {
            return Mutex.TryOpenExisting(Const.Lock, out var result);
        }
```

在使用 Mutex 如果没有传入 Name 那么将会在一个进程内，使用相同对象的锁，做到同步。如果给了命名，将会调用内核，在所有进程同步



使用锁判断进程存在将需要小心这些问题 [.NET 中使用 Mutex 进行跨越进程边界的同步 - walterlv](https://blog.walterlv.com/post/mutex-in-dotnet.html )

## 使用进程名判断

另一个方法是通过进程名判断，这样判断的进程就不需要是自己写的进程，通过进程名判断是获取对应进程名的进程，通过判断返回数组元素，请看代码

```csharp
        public bool FindExistByProcessName()
        {
            var name = "HacurbonefeciloQicejewarrerai";
            return Process.GetProcessesByName(name).Length > 0;
        }
```

上面代码的 name 传入需要判断的进程

在使用进程名判断的时候，可选的方法还有通过 `Process.GetProcesses()` 然后判断里面的进程名，但是使用上面方法的性能是最高的

使用 Process 判断进程是否存在的方法性能请看 [.NET 中 GetProcess 相关方法的性能 - walterlv](https://blog.walterlv.com/post/performance-of-get-process.html )

现在已经告诉大家两个方法判断进程是否存在，通过内核方式判断的性能比较快，请看下面性能

## 两个方法性能

使用[标准性能测试](https://blog.lindexi.com/post/C-%E6%A0%87%E5%87%86%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95.html ) 测试了两个方法的性能，可以看到使用内核的方式的性能很快

``` ini

BenchmarkDotNet=v0.11.5, OS=Windows 10.0.18362
Intel Core i7-6700 CPU 3.40GHz (Skylake), 1 CPU, 8 logical and 4 physical cores
  [Host]     : .NET Framework 4.7.2 (CLR 4.0.30319.42000), 64bit RyuJIT-v4.8.4010.0
  DefaultJob : .NET Framework 4.7.2 (CLR 4.0.30319.42000), 64bit RyuJIT-v4.8.4010.0


```

|                    Method |         Mean |       Error |      StdDev |
|-------------------------- |-------------:|------------:|------------:|
|    FindExistByProcessName | 6,955.411 us | 197.9235 us | 580.4753 us |
| FindNotExistByProcessName | 6,552.935 us | 198.3320 us | 271.4790 us |
|          FindExistByMutex |     3.032 us |   0.0908 us |   0.2649 us |
|       FindNotExistByMutex |     2.064 us |   0.0412 us |   0.0521 us |

测试代码请看下面

```csharp
    public class Program
    {
        static void Main(string[] args)
        {
            BenchmarkRunner.Run<Program>();
        }

        [Benchmark]
        public bool FindExistByProcessName()
        {
            var name = "HacurbonefeciloQicejewarrerai";
            return Process.GetProcessesByName(name).Length > 0;
        }

        [Benchmark]
        public bool FindNotExistByProcessName()
        {
            return Process.GetProcessesByName("不存在的进程").Length > 0;
        }

        [Benchmark]
        public bool FindExistByMutex()
        {
            return Mutex.TryOpenExisting(Const.Lock, out var result);
        }

        [Benchmark]
        public bool FindNotExistByMutex()
        {
            return Mutex.TryOpenExisting("不存在的进程", out var result);
        }
    }
```

在运行测试代码之前先使用下面代码测试判断进程存在

```csharp
        static void Main(string[] args)
        {
            Process.Start("HacurbonefeciloQicejewarrerai.exe");

            var program = new Program();

            Console.WriteLine($"FindExistByProcessName={program.FindExistByProcessName()}");
            Console.WriteLine($"FindNotExistByProcessName={program.FindNotExistByProcessName()}");
            Console.WriteLine($"FindExistByMutex={program.FindExistByMutex()}");
            Console.WriteLine($"FindNotExistByMutex={program.FindNotExistByMutex()}");
        }
```

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/321deaa01e76f3b3a21271e7200e6ecc0529587e/HacurbonefeciloQicejewarrerai) 欢迎下载

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
