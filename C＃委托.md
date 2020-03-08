# C＃委托

关于什么是委托，委托如何使用，我在这里就不说了。

需要说的：

1. 委托是函数指针链

2. 委托的 BeginInvoke

3. 委托如果出现异常，会如何

<!--more-->
<!-- CreateTime:2019/9/2 12:57:37 -->


<div id="toc"></div>

如果不知道函数指针，可以继续往下看，我来告诉大家，为何需要委托。

假如在写游戏，有一个人物，他会升级，那么在他升级的时候，需要给他添加潜力值，而判断升级是经验类需要写的。

先写一个经验类。


```csharp
    核心
    如果当前经验大于最大经验，升级。
```

但是升级是在经验类里，经验类外的人物不会知道已经升级了，那么如何让人物知道升级？

这时可以使用委托。


```csharp
     如果当前经验大于最大经验
     调用函数升级
```

那么函数 升级 如何让人物知道，可以使用一个委托


```csharp
    升级
    调用委托
```

于是人物可以添加函数到经验。


```csharp
    经验.升级=人物.升级
```

这样，经验的升级就是使用人物的升级，可以使用其他类的函数。

上面没有代码，现在来说个有代码的。

假如需要去寄快递，那么需要找邮递，把东西给他

假设有个邮递，可以寄快递


```csharp
    class 邮递
    {

        public static Action 寄快递;
    }
```

委托最好用Action

这里定义委托是，不知道会是哪个快递员会去寄快递，在实际，谁也不知道最后是哪个，快递员可能说不想干了，实际快递太辛苦，大家多体谅。所以不能写函数说，快递A 给 小明去寄，因为不知道小明是不是今天上班。

来写一个 主角 张 ，他需要去寄快递。

```csharp
    class MrZhang
    {
         public void 寄快递()
         {
         	邮递.寄快递();
         }

    }
```

不知道是谁上班，但是我可以寄快递。

最后，今天来上班的快递员



```csharp

    快递员 a = new 快递员();
    邮递.寄快递 = a.寄快递;

    class 快递员
    {

         public void 寄快递()
         {
              
         }
    }
```

就是快递，所以这就是委托使用。

使用一个函数，不知道他是谁用的，可以使用委托。






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

本文内容属于大量抄袭，代码是自己写，但是抄了代码大神写的，于是羞愧放下他的博客：

http://www.cnblogs.com/free722/archive/2011/04/04/2005275.html

http://stackoverflow.com/questions/25979264/understanding-the-wpf-dispatcher-begininvoke


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  