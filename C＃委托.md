# C＃委托

关于什么是委托，委托如何使用，我在这里就不说了。

需要说的：

1. 委托是函数指针链

2. 委托的 BeginInvoke

3. 委托如果出现异常，会如何

<!--more-->
<!-- csdn -->

如果使用的委托是 多播，那么小心用 BeginInvoke 


如果使用 同步，那么多个方法是可以运行

```csharp
            static void Main(string[] args)
        {
            Action m = M1;
            m += M2;
            m.Invoke();
        }

        private static void M1()
        {
            p();
        }

        private static void M2()
        {
            p();
        }

        private static void p()
        {
            Console.WriteLine("线程ID为：{0}", Thread.CurrentThread.ManagedThreadId);
        }
```


如果使用的BeginInvoke，那么在委托多播使用 BeginInvoke 会报异常 `System.ArgumentException:“The delegate must have only one target.”`

在一个线程不能同时执行多个方法，使用 BeginInvoke 必须在多个线程，所以可以使用 获得所有方法，然后执行


```csharp
        static void Main(string[] args)
        {
            Action m = M1;
            m += M2;
            //m.Invoke();
            List<IAsyncResult> list=new List<IAsyncResult>();
            var invtl = m.GetInvocationList().OfType<Action>().ToList();
            foreach (Action temp in invtl)
            {
                list.Add(temp.BeginInvoke((e) => { }, null));
            }

            for (int i = 0; i < invtl.Count(); i++)
            {
                invtl[i].EndInvoke(list[i]);
            }
           
        }

        private static void M1()
        {
            p();
        }

        private static void M2()
        {
            p();
        }

        private static void p()
        {
            Console.WriteLine("线程ID为：{0}", Thread.CurrentThread.ManagedThreadId);
        }
```
有两个需要知道：

`m.GetInvocationList()` 得到 Delegate 不能直接执行。

需要转类型，定义是知道他是什么类型，上面使用的是 Action ，所以可以转 Action ，如果是自定义的，那么使用自定义的。

第二，如果使用 BeginInvoke ，在等待时，需要拿到 IAsyncResult 才可以。


```csharp
    var temp = m.BeginInvoke((e) => { }, null);
    m.EndInvoke(temp);
```

上面代码：如果要使用 BeginInvoke ，小心  Delegate 是多个函数，不是一个函数，所以使用时，建议使用


```csharp
            var invtl = m.GetInvocationList().OfType<Action>().ToList();
            foreach (Action temp in invtl)
            {
                list.Add(temp.BeginInvoke((e) => { }, null));
            }
```

需要把类型换自己的类型。

## 如果委托发生异常

如果委托方法里没有 try catch，那么如果使用 invoke， 委托是函数指针链，所以会在执行异常退出。

如果使用 BeginInvoke ，那么会在 EndInvoke 退出。


```csharp
        static void Main(string[] args)
        {
            Action m = M1;
            m += M2;
            //m.Invoke();
            List<IAsyncResult> list=new List<IAsyncResult>();
            var invtl = m.GetInvocationList().OfType<Action>().ToList();
            foreach (Action temp in invtl)
            {
                list.Add(temp.BeginInvoke((e) => { }, null));
            }

            //for (int i = 0; i < invtl.Count(); i++)
            //{
            //    invtl[i].EndInvoke(list[i]);
            //}
           
        }

        private static void M1()
        {
            p();
            throw new ArgumentException();
        }

        private static void M2()
        {
            p();
            throw new ArgumentException();
        }

        private static void p()
        {
            Console.WriteLine("线程ID为：{0}", Thread.CurrentThread.ManagedThreadId);
        }
```

上面代码没有 EndInvoke 不会有异常，异常发生在别线程，所以不会在Main函数异常。

如果使用 EndInvoke 会在 Main 异常


```csharp
           static void Main(string[] args)
        {
            Action m = M1;
            m += M2;
            //m.Invoke();
            List<IAsyncResult> list=new List<IAsyncResult>();
            var invtl = m.GetInvocationList().OfType<Action>().ToList();
            foreach (Action temp in invtl)
            {
                list.Add(temp.BeginInvoke((e) => { }, null));
            }

            for (int i = 0; i < invtl.Count(); i++)
            {
                invtl[i].EndInvoke(list[i]);
            }

        }

        private static void M1()
        {
            p();
            throw new ArgumentException();
        }

        private static void M2()
        {
            p();
            throw new ArgumentException();
        }

        private static void p()
        {
            Console.WriteLine("线程ID为：{0}", Thread.CurrentThread.ManagedThreadId);
        }
```

## 事件

如果事件使用 BeginInvoke 和 委托一样


```csharp
     
            _event += (s, e) => M1();
            _event += (s, e) => M2();
            _event?.BeginInvoke(null, null, (e) => { }, null);

     private static EventHandler _event;
```

异常：`System.ArgumentException:“The delegate must have only one target.”`

需要和 委托 一样，获得函数指针，执行，如果方法里有异常，那么不会在 Main 方法异常，如果没有使用 End 。

```csharp
            foreach (var temp in _event.GetInvocationList().OfType<EventHandler>())
            {
                temp.BeginInvoke(null, null, (e) => { }, null);
            }
```

本文还没写好，如果有问题，请联系我

http://www.cnblogs.com/free722/archive/2011/04/04/2005275.html

http://stackoverflow.com/questions/25979264/understanding-the-wpf-dispatcher-begininvoke


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  