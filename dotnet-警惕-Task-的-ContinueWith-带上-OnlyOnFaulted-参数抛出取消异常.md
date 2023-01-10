
# dotnet 警惕 Task 的 ContinueWith 带上 OnlyOnFaulted 参数抛出取消异常

本文记录 dotnet 的一个令人迷惑的设计，在 Task 里，有一个叫 ContinueWith 的方法，此方法可以在 Task 完成时执行传入的委托。在 ContinueWith 方法里面，还有一个可选的 TaskContinuationOptions 参数，在此参数里面传入 OnlyOnFaulted 即可在 Task 出错时才执行传入的委托，然而此行为迷惑的是在 Task 正在执行完成却抛出取消异常

<!--more-->


<!-- CreateTime:2022/12/21 8:37:37 -->


<!-- 发布 -->

在等待任务执行完成之后，干某个活的事情上，有多个可选方法。一个就是老实使用 await 等待 Task 执行完成，然后再继续编写后续逻辑，如以下代码

```csharp
await task;
干自己的活();
```

另一个方法就是通过 ContinueWith 方法，比如在使用 Task.Run 执行某个 Foo 方法之后，再 干自己的活 的代码

```csharp
        var task = Task.Run(Foo).ContinueWith(t =>
        {

        });
```

以上的 ContinueWith 方法里面传入的委托是不管 Task 的执行状态，无论是成功还是失败都能进入。如果只期望只有在失败时才进入，可以传入 OnlyOnFaulted 参数，代码如下

```csharp
        var task = Task.Run(Foo).ContinueWith(t =>
        {

        }, TaskContinuationOptions.OnlyOnFaulted);
```

然而这里存在一个令人迷惑的行为，大家猜猜，当 Foo 正常执行时，等待上面代码的 task 时，是否会抛出异常

答案是抛出 TaskCanceledException 异常

```csharp
        var task = Task.Run(Foo).ContinueWith(t =>
        {

        }, TaskContinuationOptions.OnlyOnFaulted);

        try
        {
            await task;
        }
        catch (TaskCanceledException e)
        {

        }

    static void Foo()
    {
    	
    }
```

这是因为 dotnet 认为 ContinueWith 里面的委托被取消了

那如果 Task 执行过程中抛出异常呢？看看下面的代码

```csharp
        var task1 = Task.Run(FooWithException).ContinueWith(t =>
        {

        }, TaskContinuationOptions.OnlyOnFaulted);

        await task1;

    static void FooWithException()
    {
        throw new Exception("lindexi is doubi");
    }
```

可以看到 task1 正常被等待，啥事都没有发生

这么特别诡异起来了，很好就在代码里面挖坑。毕竟写了以上代码的开发者更多的是进行测试 Task 异常的情况。再加上如果偶尔的正常执行完成，抛出的是取消异常，很多开发者都会以为是正常被取消而已

也有伙伴说，那分开两个 Task 等待好了，如以下代码

```csharp
        var task = Task.Run(Foo);

        var task1 = task.ContinueWith(t =>
        {

        }, TaskContinuationOptions.OnlyOnFaulted);
```

但是以上代码解决不了的问题是，如果期望等待整个大的 Task 执行完成，也就是 Task 和 ContinueWith 里面的内容全部执行完成，那这个逻辑就诡异了

也就是只有在无需等待 ContinueWith 执行结果的情况下，才可以推荐使用 OnlyOnFaulted 参数。没有等待 ContinueWith 执行结果，且刚好 Task 是正常执行的，这是不会将取消异常抛到 UnobservedTaskException 里的

```csharp
        TaskScheduler.UnobservedTaskException += (sender, eventArgs) =>
        {
        };
```

在 dotnet 的设计里面，如果一个 Task 存在异常，且这个 Task 的异常没有被任何代码捕获到，将在此 Task 被 GC 时，抛到 UnobservedTaskException 里面。可以通过如上代码的事件，获取到是否存在有 Task 的异常没有被捕获。进入 UnobservedTaskException 事件的异常不会导致应用挂掉，只是用来记录日志或者埋点上报等，让开发者知道有某个 Task 的异常没有被捕获


本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/f758059f3f9f9bbfe2e7205a137d2e5b3da31f7a/HojohoyahobaWayfahurhalqeje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f758059f3f9f9bbfe2e7205a137d2e5b3da31f7a/HojohoyahobaWayfahurhalqeje) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f758059f3f9f9bbfe2e7205a137d2e5b3da31f7a
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f758059f3f9f9bbfe2e7205a137d2e5b3da31f7a
```

获取代码之后，进入 HojohoyahobaWayfahurhalqeje 文件夹

更多博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。