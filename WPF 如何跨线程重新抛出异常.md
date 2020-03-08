# WPF 如何跨线程重新抛出异常

有一些代码是在框架层写的，这部分代码不应该在后台线程抛出异常，如何将后台线程的异常在主线程抛出，同时不会带上主线程的堆栈可以保留异常的全部信息

<!--more-->
<!-- CreateTime:2019/10/7 13:24:54 -->

<!-- csdn -->

在 .NET 提供了 ExceptionDispatchInfo 类，可以用于捕获某个异常，然后使用提供的抛出方法重新抛出

可以用在某个地方统一收集异常，然后统一抛出，此时抛出的异常的堆栈和信息都不会改变，会比下面的方法更好

```csharp
        public static void NalbibechaLuhaqayna()
        {
            Exception exception = Foo();

            if (exception != null)
            {
                ReThrowException(exception);
            }
        }

        private static Exception Foo()
        {
            Exception exception = null;
            try
            {
                throw new Exception();
            }
            catch (Exception e)
            {
                exception = e;
            }

            return exception;
        }
             
        private static void ReThrowException(Exception exception)
        {
            throw exception;
        }
```

上面代码使用 throw 在另一个函数抛出，可以从堆栈看到，没有原先抛异常 Foo 函数，将会让异常堆栈加上了抛出函数的调用堆栈，如果此时是在跨线程用的，那么将会找不到原有线程堆栈

```
   at KicaicicayiJearjelrelur.MainWindow.ReThrowException(Exception exception)
   at KicaicicayiJearjelrelur.MainWindow.NalbibechaLuhaqayna()
   at KicaicicayiJearjelrelur.MainWindow..ctor()
```

这样进行异常调试就比较难知道是在哪个函数的异常，特别是空异常

如果使用下面方法抛出，那么可以保存异常堆栈

```csharp
        private static void ReThrowException(Exception exception)
        {
            ExceptionDispatchInfo.Capture(exception).Throw();
        }
```

可以从调用堆栈看到开始的异常的堆栈

```
   at KicaicicayiJearjelrelur.MainWindow.Foo()
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at KicaicicayiJearjelrelur.MainWindow.ReThrowException(Exception exception)
   at KicaicicayiJearjelrelur.MainWindow.NalbibechaLuhaqayna()
   at KicaicicayiJearjelrelur.MainWindow..ctor()
```

如果是在后台线程框架抛出的，建议放在主线程可以这样做

```csharp
        private void ReThrowException(Exception exception)
        {
            Dispatcher.InvokeAsync(() => { ExceptionDispatchInfo.Capture(exception).Throw(); });
        }
```

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b418b8d562a9534b450488aa15e8ae58bd54542e/KicaicicayiJearjelrelur/KicaicicayiJearjelrelur) 欢迎下载

在 WPF 框架我使用这个方法提了建议，在触摸线程的异常抛到主线程，如果您觉得我的方法可以，请在 [这里](https://github.com/dotnet/wpf/pull/945 ) 点赞

详细请看 [使用 ExceptionDispatchInfo 捕捉并重新抛出异常 - walterlv](https://blog.walterlv.com/post/exceptiondispatchinfo-capture-throw.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
