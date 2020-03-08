# C# 设计模式 责任链

责任链模式是一种对象的行为模式。在责任链模式里，很多对象由每一个对象对其下家的引用而连接起来形成一条链。请求在这个链上传递，直到链上的某一个对象决定处理此请求。发出这个请求的客户端并不知道链上的哪一个对象最终处理这个请求，这使得系统可以在不影响客户端的情况下动态地重新组织和分配责任。《JAVA与模式》

<!--more-->
<!-- CreateTime:2019/9/2 12:57:37 -->


<div id="toc"></div>

我们在 C# 也可以使用责任链。

首先我们需要一个接口`IHandle`接受我们的责任，在里面，最简单的责任链只有 Successor 和 Request AddSuccessor ，请看代码 

```csharp
    public interface IHandle
    {
        IHandle Successor
        {
            set;
            get;
        }

        void Request(string str);

        void AddSuccessor(IHandle successor);
    }
```

责任链的下一个责任接受：Successor

处理责任：Request

添加处理责任的下一个：AddSuccessor

然后我们需要一个实际处理类，这个类集成接口`IHandle`。可以看到我的接口`IHandle `只是处理字符串，其实我们可以处理很多的，但是为了简单，我们就先写字符串。

假如我们是员工，发起的请求是叫老板加工资，那么开始决定工资还不是他，需要经过主管、HR、然后是老板，他们组成了一个责任链。

首先我们定义员工，他可以发送出责任需要让我们的具体处理者处理，但是我们这时看到了主管等其实有重复的，如果主管不同意处理，那么就没必要进行HR同意，所以我们的`IHandle`有`AreHandle`是不是被处理，因为无论是不是被处理还是要传给下一个，他需要决定这个处理我要不要继续，如果我们的员工提出要求，主管看他不爽，每次都不给他提工资，而我们HR也因为主管处理了，他就不接受，那么这样，员工就得不到提工资，为了解决，我们让HR也要接收到，也就是不管如何都把这个传到下一层，当然这样就需要传下一个`AreHandle`。如果觉得不需要，那么就不用传下。

另一个，我们不知道我们什么时候实现主管、什么时候实现HR，我们责任链的位置重要，因为如果主管处理了，HR就可以不处理，那么我们需要顺序，一般在添加那里加上权限，这里写`Access`。我们在添加具体处理，一般判断我们的下一个处理是否存在，如果不存在，直接添加输入参数下一个处理，如果存在，判断权限大小，如果比他大就代换他，如果比他小，就给下一个处理。

```csharp
        public void AddSuccessor(IHandle successor)
        {
            if (Successor == null)
            {
                Successor = successor;
            }
            else
            {
                if (successor.Access > Successor.Access)
                {
                    var temp = Successor;
                    Successor = successor;
                    successor.Successor = temp;
                }
                else
                {
                    Successor.AddSuccessor(successor);
                }
            }
        }
```

我们发现如果把具体处理写为一个类，然后实现，这样每次使用函数`Request`都用成员，这样不太好，因为我们有功能扩展，所以我们把`ConcreteHandler`写为抽象，然后继承，写出主管等的类。

```csharp
    public class GHandle : ConcreteHandler
    {
        public override void Request(string str)
        {
            NotifyProperty.Reminder?.Invoke("经理处理"+str);
            Successor?.Request(str);
        }
    }

    public class DeptHandle : ConcreteHandler
    {
        public override void Request(string str)
        {
            NotifyProperty.Reminder?.Invoke("主管处理"+str);
            Successor?.Request(str);           
        }
    }
```


## 后退按钮使用责任链

我看到堆栈炸了有人问我，为什么一按后退就炸。

我看了他的源代码，他每个页面都把后退按钮点击事件+=他的方法。

我们可以使用UWP的后退按钮，但是需要小心，在哪些处理需要知道，不可以在每个需要处理都添加事件。

那么如何添加后退按钮，才可以在需要后退的时候进行后退，可以用到上面说的设计，添加一个链，需要做一个类，如果直接写，看起来比较难。

新建一个类，这个类用做责任，通过这个类，可以做 MVVM ，如果对于这个不熟，请看 [win10 uwp MVVM入门](https://blog.lindexi.com/post/win10-uwp-MVVM%E5%85%A5%E9%97%A8.html)


本文告诉大家如何做出双击退出应用。

首先需要创建两个类作为责任链，请看下面。

```csharp
  /// <summary>
    /// 责任链模式
    /// </summary>
    public class FjyhtrOcbhzjwi
    {
        public static FjyhtrOcbhzjwi Fhnazmoul { get; } = new FjyhtrOcbhzjwi();

        public bool Handle { get; set; }

        public void AddSuccessor(AjuvqrDqsoljna ajuvqrDqsoljna)
        {
            AjuvqrDqsoljna.AddSuccessor(ajuvqrDqsoljna);
        }

        public void RemoveSuccessor(AjuvqrDqsoljna ajuvqrDqsoljna)
        {
            AjuvqrDqsoljna.RemoveSuccessor(ajuvqrDqsoljna);
        }

        public void Request()
        {
            Handle = false;
            AjuvqrDqsoljna.Request(this);
        }

        private IHandle AjuvqrDqsoljna { set; get; } = new AjuvqrDqsoljna(fjyhtrOcbhzjwi => { });
    }

    public interface IHandle
    {
        IHandle Successor
        {
            set;
            get;
        }

        int Hnkdqckyr { get; }

        void Request(FjyhtrOcbhzjwi fjyhtrOcbhzjwi);

        void AddSuccessor(IHandle ajuvqrDqsoljna);
        void RemoveSuccessor(IHandle ajuvqrDqsoljna);
    }

    public class AjuvqrDqsoljna : IHandle
    {
        public AjuvqrDqsoljna(Action<FjyhtrOcbhzjwi> request)
        {
            Request = request;
        }

        /// <summary>
        /// 权限
        /// </summary>
        public int Hnkdqckyr { set; get; }

        public Action<FjyhtrOcbhzjwi> Request { get; }

        IHandle IHandle.Successor { get; set; }

        void IHandle.Request(FjyhtrOcbhzjwi fjyhtrOcbhzjwi)
        {
            Request.Invoke(fjyhtrOcbhzjwi);
            ((IHandle) this).Successor?.Request(fjyhtrOcbhzjwi);
        }

        void IHandle.AddSuccessor(IHandle successor)
        {
            if (((IHandle)this).Successor == null)
            {
                ((IHandle) this).Successor = successor;
            }
            else
            {
                if (successor.Hnkdqckyr > ((IHandle) this).Successor.Hnkdqckyr)
                {
                    var temp = ((IHandle) this).Successor;
                    ((IHandle) this).Successor = successor;
                    successor.Successor = temp;
                }
                else
                {
                    ((IHandle) this).Successor.AddSuccessor(successor);
                }
            }
        }

        void IHandle.RemoveSuccessor(IHandle ajuvqrDqsoljna)
        {
            if (ajuvqrDqsoljna == null)
            {
                return;
            }
            if (((IHandle) this).Successor == ajuvqrDqsoljna)
            {
                ((IHandle) this).Successor = ((IHandle) this).Successor.Successor;
            }
            else
            {
                ((IHandle) this).Successor?.RemoveSuccessor(ajuvqrDqsoljna);
            }
        }
    }
```

在使用的时候，通过调用`FjyhtrOcbhzjwi`就可以获得插入新的处理。接下来就是需要返回的按钮，参见[win10 UWP 标题栏后退](https://blog.lindexi.com/post/win10-UWP-%E6%A0%87%E9%A2%98%E6%A0%8F%E5%90%8E%E9%80%80.html )

```csharp
        protected override void OnNavigatedTo(NavigationEventArgs e)
        {
            base.OnNavigatedTo(e);
            //启动后退关闭
            Windows.UI.Core.SystemNavigationManager.GetForCurrentView().AppViewBackButtonVisibility = Windows.UI.Core.AppViewBackButtonVisibility.Visible;
            Windows.UI.Core.SystemNavigationManager.GetForCurrentView().BackRequested += BackRequested;
            FiontwzNdqd();
        }
```

可以看到`FiontwzNdqd`函数就是做双击退出的，但是双击退出需要一个字段记录是否退出。

```csharp
        private bool _ajuvqrDqsoljnaMkfsjNiydfobt;

```

然后就是添加双击退出的函数

```csharp
        private void FiontwzNdqd()
        {
            var ajuvqrDqsoljna = new AjuvqrDqsoljna(fjyhtrOcbhzjwi =>
            {
                //双击退出
                if (fjyhtrOcbhzjwi.Handle)
                {
                    _ajuvqrDqsoljnaMkfsjNiydfobt = false;
                    return;
                }
                if (_ajuvqrDqsoljnaMkfsjNiydfobt)
                {
                    Application.Current.Exit();
                }
                else
                {
                    LyfxkdxmSzjd.Text= "再次点击退出";
                    _ajuvqrDqsoljnaMkfsjNiydfobt = true;
                }
            });
            FjyhtrOcbhzjwi.Fhnazmoul.AddSuccessor(ajuvqrDqsoljna);
        }
```

但是这样写没有动画，于是添加动画，有关动画，参见：[win10 UWP 动画](https://blog.lindexi.com/post/win10-UWP-%E5%8A%A8%E7%94%BB.html )

```csharp
        private void ViewModel_LyfxkdxmSzjd(object sender, string e)
        {
            LyfxkdxmSzjd.Text = e;
            LyfxkdxmSzjd.Visibility = Visibility;
            LyfxkdxmSzjd.Opacity = 1;

            Storyboard sb = new Storyboard();
            DoubleAnimation animation = new DoubleAnimation
            {
                From = 1,
                To = 0,
                Duration = new Duration(TimeSpan.FromSeconds(2))
            };
            Storyboard.SetTargetName(animation, nameof(LyfxkdxmSzjd));
            Storyboard.SetTargetProperty(animation, "Opacity");
            sb.Children.Add(animation);
            sb.Completed += (o, ee) =>
            {
                LyfxkdxmSzjd.Visibility = Visibility.Collapsed;
                _ajuvqrDqsoljnaMkfsjNiydfobt = false;
            };
            Storyboard.SetTarget(animation, LyfxkdxmSzjd);
            sb.Begin();
        }
```

这里的动画这样写是因为在 ViewModel 有一个事件，这个事件就是通知，于是就需要添加事件，在界面显示。刚好在显示结束的时候关闭双击退出。

在我之前写的游戏[win10 uwp 商业游戏](https://blog.lindexi.com/post/win10-uwp-%E5%95%86%E4%B8%9A%E6%B8%B8%E6%88%8F.html )进入游戏时，用户按下返回按钮，需要返回欢迎界面，那么这时候就需要添加后退的处理。

因为我添加的是 MVVM 框架，于是在跳转进游戏的 ViewModel 时添加处理。关于这个框架，请看[win10 uwp MVVM 轻量框架](http://blog.csdn.net/lindexi_gd/article/details/78083924 )这里，但是我不会在本文用了太多这个框架的东西。在没有看这篇文章还是可以继续阅读。

```csharp
        public override void OnNavigatedTo(object sender, object obj)
        {
            //省略其他的代码

            _ajuvqrDqsoljna = _ajuvqrDqsoljna ?? new AjuvqrDqsoljna(async fjyhtrOcbhzjwi =>
            {
                if (fjyhtrOcbhzjwi.Handle)
                {
                    return;
                }
                fjyhtrOcbhzjwi.Handle = true;
              
                //游戏保存
                await AccountGoverment.JwAccountGoverment.Storage();
                //返回上一层
                Send(new BackTvvxwlwIlibbcpMessage(this));

                //fjyhtrOcbhzjwi.Handle = true; 不要写在后面

            }){
                   Hnkdqckyr = 10
              };
            FjyhtrOcbhzjwi.Fhnazmoul.AddSuccessor(_ajuvqrDqsoljna);
        }
```

上面代码主要是添加在后退时，保存游戏和返回到上一层，代码最重要的是使用`fjyhtrOcbhzjwi.Handle = true`，于是在他后面的处理就可以知道自己需要或不需要处理。

当然自己添加的处理也是需要判断当前是否已经有权限比他高的进行处理，如果有，就不处理。这样写就可以在游戏进行返回。

上面代码用到框架只有一句`Send(new BackTvvxwlwIlibbcpMessage(this))` 他可以让页面返回上一页，只需要发送消息，不需要知道如何去做。

需要知道的是关于 async 可能出现一个问题，请看代码，最后我去掉了`fjyhtrOcbhzjwi.Handle`，说不要写在后面。这里的意思是如果调用一个方法，这个方法有 await 那么这个方法如果不是 async Task 那么就会直接被跳过，不会往下面执行，所以如果把最后一句代码替换前面的，那么就会调用责任链的下一处理，但是没有告诉他已经处理了。所以在责任链，需要注意同步和异步的转换，如果实在需要，那么请参见我的博客，如何把异步转同步。


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。