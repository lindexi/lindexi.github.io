
# C＃ await 高级用法

本文告诉大家 await 的高级用法，包括底层原理。

<!--more-->


<!-- csdn -->

昨天看到[太子](https://walterlv.github.io/)写了一段代码，我开始觉得他修改了编译器，要不然下面的代码怎么可以编译通过

```csharp
await "林德熙逗比";
```

需要知道，基本可以添加 await 都是可以等待的类型，如 Task 。如果一个类需要可以被等待，那么这个类必须满足以下条件

 - 类里有一个 GetAwaiter 函数

 - GetAwaiter 有返回值，返回值需要继承 INotifyCompletion 并且有 `bool IsCompleted { get; }`,`GetResult()`,`void OnCompleted(Action continuation)` 定义

参见：[如何实现一个可以用 await 异步等待的 Awaiter - walterlv](https://walterlv.github.io/post/write-custom-awaiter.html )

但是上面的代码使用的是一个字符串，什么时候可以修改继承字符串？

先让我来说下 await 原理，因为知道了原理，上面的代码实现很简单。看完了本文，你就会知道如何让几乎所有类型包括 int 、string 、自定义的类都支持 await 。

如果真的不想看原理，那么请直接调到文章的最后，看到最后很快就知道如何做。

## 原理

在 .net 4.5 之后，框架默认提供 async 和 await 的语法糖，这时千万不要认为进入 await 就会进入一个新的线程，实际上不一定会进入一个新的线程才会调用 await 。

那么 await 的语法糖写的是什么？实际上就是以前的 Begin xx 和 End xx 的语法糖。

古时候的写法：

```csharp
foo.Beginxx();

foo.Endxx(传入委托);
```

这样大家就无法在一个流程写完，需要分为两个东西，而在 Continus with 下，就需要传入委托。如果委托里又使用了异步，那么又需要传入委托

```csharp
       task.ContinueWith(_ =>
            {
                Task t1 = new Task(() => { });
                t1.ContinueWith((t2) =>
                {
                    //可以看到如果进入很多的委托
                });
            });
```

所以这时就使用了 await ，可以让大家按照顺序写。

```csharp
await task;
Task t1 = new Task(() => { });
await t1;
//可以看到这时不需要进入委托
```

实际上 await 是在编译时支持的，请看[进阶篇：以IL为剑，直指async/await - 布鲁克石 - 博客园](http://www.cnblogs.com/brookshi/p/5240510.html )

而且千万不要认为 await 一定会进入一个新的线程，实际上他只是把需要写在多处的代码，可以按照流写下载，和写同步代码一样。如果感兴趣 await 不一定会进入一个新的线程请看 [There Is No Thread](http://blog.stephencleary.com/2013/11/there-is-no-thread.html )

## 使用

因为 await 需要找到一个 GetAwaiter 函数，这个函数即使是扩展方法也可以，所以其实上面的代码是这样写的

```csharp

    public static class KvpbamjhKsvm
    {
        public static HeabdsdnbKevx GetAwaiter(this string str)
        {
            return new HeabdsdnbKevx();
        }
    }

    public class HeabdsdnbKevx : INotifyCompletion
    {
        public bool IsCompleted { get; }

        public void GetResult()
        {
        }

        /// <inheritdoc />
        public void OnCompleted(Action continuation)
        {
        }
    }
```

HeabdsdnbKevx 就是一个可以等待的类型

现在就可以写出下面的代码

```csharp
        private static void Main(string[] args)
        {
            DdngSiwchjux();
        }

        private static async void DdngSiwchjux()
        {
            await "林德熙逗比";
        }
```

当然，上面的这个代码可以运行，不过不会返回什么。下面让我加上一句代码。

```csharp
        private static void Main(string[] args)
        {
            DdngSiwchjux();
        }

        private static async void DdngSiwchjux()
        {
            await "林德熙逗比";
            Console.WriteLine("csdn");
        }
```

这时可以看到，`Console.WriteLine("csdn");`不会运行，因为这时如果在 `OnCompleted` 函数打断点就可以看到，执行`await "林德熙逗比"`之后就进入`OnCompleted` 函数。从上面的原理可以知道，这个函数传入的参数就是两个`await`或 `await`和函数结束之间的代码。如果需要让`Console.WriteLine("csdn");`运行，那么只需要在`OnCompleted`运行参数

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018211153416.jpg)

```csharp
   public void OnCompleted(Action continuation)
        {
            continuation();
        }
```

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018211154415.jpg)

但是作为一个挖坑专业的大神，怎么可以就扩展 string ，下面我把 int 进行扩展

```csharp
    public static class KvpbamjhKsvm
    {
        public static HeabdsdnbKevx GetAwaiter(this int dxpbnzSce)
        {
            return new HeabdsdnbKevx();
        }
    }
```

随意写一个值，然后进行等待

![](http://7xqpl8.com1.z0.glb.clouddn.com/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2018211154944.jpg)

现在我准备在 object 加一个扩展方法，所有类型都可以等待，然后把这个扩展方法的 namespace 写为 System ，这样大家就不知道这个是我写的，过了一年我就告诉大家这是 C# 的特性，所有的类都可以等待。但是这个特性需要开光才可以使用，你们直接建的项目没有开光所以没法使用这个特性。

## 相关博客

[使用 Task.Wait()？立刻死锁（deadlock） - walterlv](https://walterlv.github.io/post/deadlock-in-task-wait.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。