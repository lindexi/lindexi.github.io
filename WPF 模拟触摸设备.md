# WPF 模拟触摸设备

在 WPF 中触摸只是框架的一层，可以通过代码模拟触摸

<!--more-->
<!-- CreateTime:2019/11/29 8:47:53 -->

<!-- csdn -->

创建一个类继承 TouchDevice 然后重写 GetTouchPoint 和 GetIntermediateTouchPoints 方法，可以在这个类里面通过调用 ReportDown 等方法模拟触摸的按下和移动

最简单的实现请看下面代码

```csharp
    public class BurnerkadelWallnadarli : TouchDevice
    {
        /// <inheritdoc />
        public BurnerkadelWallnadarli(int deviceId, Window window) : base(deviceId)
        {
            Window = window;
        }

        /// <summary>
        /// 触摸点
        /// </summary>
        public Point Position { set; get; }

        /// <summary>
        /// 触摸大小
        /// </summary>
        public Size Size { set; get; }

        public void Down()
        {
            TouchAction = TouchAction.Down;

            if (!IsActive)
            {
                SetActiveSource(PresentationSource.FromVisual(Window));

                Activate();
                ReportDown();
            }
            else
            {
                ReportDown();
            }
        }

        public void Move()
        {
            TouchAction = TouchAction.Move;

            ReportMove();
        }

        public void Up()
        {
            TouchAction = TouchAction.Up;

            ReportUp();
            Deactivate();
        }


        private Window Window { get; }

        private TouchAction TouchAction { set; get; }

        /// <inheritdoc />
        public override TouchPoint GetTouchPoint(IInputElement relativeTo)
        {
            return new TouchPoint(this, Position, new Rect(Position, Size), TouchAction);
        }

        /// <inheritdoc />
        public override TouchPointCollection GetIntermediateTouchPoints(IInputElement relativeTo)
        {
            return new TouchPointCollection()
            {
                GetTouchPoint(relativeTo)
            };
        }
    }

```

在 TouchDevice 里面，调用触摸按下和移动等的方法是没有传入参数的，在框架是通过 GetTouchPoint 拿到当前用户触摸的点

在按下的时候需要激活，激活的时候需要传入一个 PresentationSource 在框架通过这个值进行命中测试找到触摸按下的点是按到哪个元素

使用的时候只需要创建 BurnerkadelWallnadarli 然后调用对应的按下移动等方法就可以了，因为在构造的时候传入了窗口，所以在按下等事件可以通过传入的窗口进行命中测试找到按下的元素，从元素触发路由事件

大概的调用顺序是这样的，在触摸的第一个事件是按下，在按下的时候使用下面代码

```csharp
SetActiveSource(PresentationSource.FromVisual(Window));

Activate();
ReportDown();
```

在 SetActiveSource 会将传入的 PresentationSource 设置在本地的字段 `_activeSource` 这样可以在拿到点的时候调用

在调用 Activate 方法会调用一次 UpdateDirectlyOver 这个方法调用 GetTouchPoint 传入一个空参数，用来判断当前是否命中到元素，一般都是没有命中的，需要到 ReportDown 的时候才可能命中元素。在 Activate 会将当前的 TouchDevice 加入到 `TouchDevice._activeDevices` 这个静态字段里面，如果刚好这时的静态字段只有一个元素，那么就设置当前的触摸设备是主触摸设备

设置触摸设备是主触摸设备是因为在触摸的时候如果用户是多个手指触摸，一个手指对应一个触摸设备，所以第一个手指对应主触摸设备

调用 ReportDown 会先设置本地字段 `_isDown` 为 true 然后调用 UpdateDirectlyOver 方法更新当前按下点到的元素，然后调用 RaiseTouchDown 方法在当前点到的元素触摸触摸按下的路由事件，可以看到此时的路由事件是不需要再获取当前的触摸点，因为只是在点到的元素触摸事件，如果这个元素需要知道当前的触摸点，只需要在方法使用参数的 `e.GetTouchPoint` 方法就可以重新拿到触摸点。因为获取触摸点方法是可以重写的，所以第一次获取的用于命中测试的触摸点可以和元素收到触摸事件获取的触摸点返回不同的点

只需要拿到了对应的元素就可以在元素触发事件，从触摸到事件请看[WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )

调用 ReportMove 移动的方法也是差不多，首先通过 UpdateDirectlyOver 找到命中测试的元素，然后触发路由事件。如果元素不关注触摸点击的点就不需要再次调用获取触摸点方法

那么 UpdateDirectlyOver 是如何进行命中测试的？首先通过获取触摸点方法拿到传入空参数时的触摸点，这时相对的应该是窗口的坐标。通过 TouchDevice.LocalHitTest 方法拿到命中测试的元素，在底层调用的是 MouseDevice.LocalHitTest 方法

所以可以通过上面定义的类模拟触摸，只需要创建出来，然后调用对应的方法就可以，如下面的代码就模拟了按下和移动

```csharp
var burnerkadelWallnadarli = new BurnerkadelWallnadarli(1, this);

await Task.Delay(1000);
burnerkadelWallnadarli.Down();

await Task.Delay(1000);
 // 设置当前触摸点
burnerkadelWallnadarli.Move();

```

通过这个方法模拟触摸可以走原有的 WPF 触摸命中测试，也能走路由事件

关于 WPF 的触摸到事件请看 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )

本文用到的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f0f872153ed07b2141b47580a74a18a38cc56cfd/DernijacallqaNaycerejerlal)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
