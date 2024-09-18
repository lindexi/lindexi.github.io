---
title: dotnet 测试 SemaphoreSlim 的 Wait 是否保持进入等待的顺序先进先出
description: 本文记录我测试 dotnet 里面的 SemaphoreSlim 锁，在多线程进入 Wait 等待时，进行释放锁时，获取锁执行权限的顺序是否与进入 Wait 等待的顺序相同。测试的结果是 SemaphoreSlim 的 Wait 大部分情况是先进先出，按照 Wait 的顺序出来的，但是压力测试下也存在乱序，根据官方文档说明不应该依赖 SemaphoreSlim 的 Wait 做排队顺序

<!--more-->

tags: dotnet
category: 
---

<!-- CreateTime:2024/1/31 19:49:53 -->

<!-- 发布 -->
<!-- 博客 -->

根据如下的[官方文档](https://learn.microsoft.com/en-us/dotnet/api/system.threading.semaphoreslim)说明，可以看到多线程进入时是没有保证顺序出来的：

> If multiple threads are blocked, there is no guaranteed order, such as FIFO or LIFO, that controls when threads enter the semaphore.

尽管实际测试下，大部分情况都是完全按照顺序输出的，测试代码如下

```csharp
var taskList = new List<Task>();
var locker = new object();

ThreadPool.SetMinThreads(100, 100);
ThreadPool.SetMaxThreads(100,100);
var semaphore = new SemaphoreSlim(0, 1);

var autoResetEvent = new AutoResetEvent(false);

for (int i = 0; i < 100; i++)
{
    var n = i;
    taskList.Add(Task.Run(() =>
    {
        autoResetEvent.Set();

        semaphore.Wait();

        lock (locker)
        {
            Console.WriteLine(n);
        }

        semaphore.Release();
    }));

    autoResetEvent.WaitOne();
}

semaphore.Release();

Task.WaitAll(taskList.ToArray());
```

运行之后大概能看到输出是顺序的

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/a8a6fa2078b33e184c21e997956e665ddf52945b/RijallcijiDuqewerbu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/a8a6fa2078b33e184c21e997956e665ddf52945b/RijallcijiDuqewerbu) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin a8a6fa2078b33e184c21e997956e665ddf52945b
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin a8a6fa2078b33e184c21e997956e665ddf52945b
```

获取代码之后，进入 RijallcijiDuqewerbu 文件夹

由于我担心 Task 存在影响，同步也更改了 Thread 的版本，代码如下

```csharp
var taskList = new List<Thread>();
var locker = new object();

var semaphore = new SemaphoreSlim(0, 1);

var autoResetEvent = new AutoResetEvent(false);

for (int i = 0; i < 100; i++)
{
    var n = i;

    var thread = new Thread(() =>
    {
        autoResetEvent.Set();

        semaphore.Wait();

        lock (locker)
        {
            Console.WriteLine(n);
        }

        semaphore.Release();
    });

    taskList.Add(thread);
    thread.Start();

    autoResetEvent.WaitOne();
}

semaphore.Release();
```

运行以上代码，依然大部分时候看到输出都是顺序的

尽管大部分输出都是顺序的，但是好开发者是不应该依赖 Wait 能够实现先进先出的效果的

更改的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9273d7a649c656a95db608d9735be77e12d87600/RijallcijiDuqewerbu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9273d7a649c656a95db608d9735be77e12d87600/RijallcijiDuqewerbu) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 9273d7a649c656a95db608d9735be77e12d87600
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 9273d7a649c656a95db608d9735be77e12d87600
```

获取代码之后，进入 RijallcijiDuqewerbu 文件夹
