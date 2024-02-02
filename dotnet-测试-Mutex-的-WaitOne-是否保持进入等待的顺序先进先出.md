
# dotnet 测试 Mutex 的 WaitOne 是否保持进入等待的顺序先进先出

本文记录我测试 dotnet 里面的 Mutex 锁，在多线程进入 WaitOne 等待时，进行释放锁时，获取锁执行权限的顺序是否与进入 WaitOne 等待的顺序相同。测试的结果是 Mutex 的 WaitOne 是乱序的，不应该依赖 Mutex 的 WaitOne 做排队顺序

<!--more-->


<!-- CreateTime:2024/1/31 19:36:13 -->

<!-- 发布 -->
<!-- 博客 -->

以下是测试程序代码

```csharp
var taskList = new List<Task>();
var mutex = new Mutex(false);
var locker = new object();
mutex.WaitOne();

var autoResetEvent = new AutoResetEvent(false);

for (int i = 0; i < 100; i++)
{
    var n = i;
    taskList.Add(Task.Run(() =>
    {
        autoResetEvent.Set();

        mutex.WaitOne();

        lock (locker)
        {
            Console.WriteLine(n);
        }

        mutex.ReleaseMutex();
    }));

    autoResetEvent.WaitOne();
}

mutex.ReleaseMutex();
Task.WaitAll(taskList.ToArray());
```

运行之后输出是乱序。证明 Mutex 的 WaitOne 没有保证获取锁出来的顺序是按照进入的顺序的，没有保证先进先出

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/c255d512b09862d291b1a5a3fb921689b0b04a58/RijallcijiDuqewerbu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c255d512b09862d291b1a5a3fb921689b0b04a58/RijallcijiDuqewerbu) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c255d512b09862d291b1a5a3fb921689b0b04a58
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c255d512b09862d291b1a5a3fb921689b0b04a58
```

获取代码之后，进入 RijallcijiDuqewerbu 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。