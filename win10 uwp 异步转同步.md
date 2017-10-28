# win10 uwp 异步转同步

有很多方法都是异步，那么如何从异步转到同步？

<!--more-->
<div id="toc"></div>

在本文开始，我必须告诉大家，这个方法可能立即死锁，所以使用的时候需要满足下面的条件

## 使用的条件

1. 异步转同步的线程不是 UI 线程

1. 如果线程是UI线程，那么异步方法不能在另外一个线程。

看到这里也许你会疑惑，为何异步方法可以不在另一个线程？实际上对于 IO 等的异步方法，都是没有创建线程，请看[There Is No Thread](http://blog.stephencleary.com/2013/11/there-is-no-thread.html )

关于这条件是如何来的，请看[使用 Task.Wait()？立刻死锁（deadlock） - walterlv](https://walterlv.gitee.io/post/deadlock-in-task-wait.html )

## 使用方法

可以使用的方法需要获得是否有返回值，返回值是否需要。

如果需要返回值，使用`GetResults`

如从文件夹获取文件：


```csharp
                StorageFolder folder = StorageFolder.GetFolderFromPathAsync("").GetResults();

```

这是同步方法，几乎不需要做什么修改

如果是没有返回值或不需要返回值的，请看下面代码


```csharp
                StorageFolder.GetFolderFromPathAsync("").AsTask().Wait();

```

假设一个方法是没返回的，可以使用`Wait`


```csharp

            Foo().Wait();


    private async Task Foo()
```

通过这个方法就可以把异步方法转同步。

如果需要反过来，把同步转异步，可以使用 [同步方法转异步](http://blog.csdn.net/lindexi_gd/article/details/57897162 )


```csharp
            await Task.Run(() =>
            {
               写你的代码
            });
```

## 使用`Task.Wait` 时需要小心死锁

参见：[使用 Task.Wait()？立刻死锁（deadlock） - walterlv](https://walterlv.gitee.io/post/deadlock-in-task-wait.html )

value task

task.wait


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  