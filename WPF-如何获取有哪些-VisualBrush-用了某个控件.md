
# WPF 如何获取有哪些 VisualBrush 用了某个控件

我写了一个特殊的控件，我期望了解到有哪些 VisualBrush 捕获了此控件，或者说有哪些 VisualBrush 用了此控件的界面

<!--more-->



<!-- 发布 -->
<!-- 博客 -->

本文的方法需要用到反射，需要使用 WPF 框架里面没有公开的字段获取某个 Visual 控件被引用的 VisualBrush 有哪些，代码如下

```csharp
    class MyUserControl : UserControl
    {
        public bool IsInVisualBrush()
        {
            return GetVisualBrushes().Any();
        }

        private List<VisualBrush> GetVisualBrushes()
        {
            var type = typeof(Visual);
            var cyclicBrushToChannelsMapField = type.GetField("CyclicBrushToChannelsMapField", BindingFlags.Static | BindingFlags.NonPublic);
            var cyclicBrushToChannelsMap = cyclicBrushToChannelsMapField.GetValue(null);

            var getValueMethod = cyclicBrushToChannelsMap.GetType().GetMethod("GetValue");
            var cyclicBrushToChannelsMapDictionary = getValueMethod.Invoke(cyclicBrushToChannelsMap, new object[] { this });
            var dictionary = cyclicBrushToChannelsMapDictionary as IDictionary;

            var visualBrushes = dictionary?.Keys.OfType<VisualBrush>().ToList() ?? new List<VisualBrush>(0);
            return visualBrushes;
        }
    }
```

通过上面代码不仅可以获取某个控件，是否被作为 VisualBrush 的 Visual 作为画刷，还可以获取当前有哪些 VisualBrush 捕获了这个控件

写一个简单的界面，将这个控件设置为某个 VisualBrush 的 Visual 内容，然后将这个 VisualBrush 作为背景

```xml
  <Grid x:Name="Grid">
    <Border x:Name="Border">
      <Border.Background>
        <VisualBrush Visual="{Binding ElementName=Container}"></VisualBrush>
      </Border.Background>
    </Border>

    <Grid x:Name="Container">
      <local:MyUserControl x:Name="MyUserControl"></local:MyUserControl>
    </Grid>
  </Grid>
```

在界面的构造里面，在 InitializeComponent 方法之后，调用 IsInVisualBrush 方法，返回的是不被 VisualBrush 捕获。但是如果在 Loaded 事件获取，返回的是没有被捕获。只有在 Loaded 事件加上 Dispatcher 延迟返回的才是被捕获

```csharp
        public MainWindow()
        {
            InitializeComponent();

            Loaded += MainWindow_Loaded;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            Dispatcher.InvokeAsync(() =>
            {
                MyUserControl.IsInVisualBrush();
            });
        }
```

而如果在点击按钮的时候，将使用了 VisualBrush 作为背景的 Border 移除，那么立刻就可以获取到没有被捕获

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            Grid.Children.Remove(Border);

            MyUserControl.IsInVisualBrush(); // 返回 false 没有被捕获
        }
```

上面代码其实用到了 WPF 的机制，在 WPF 里面，所有的控件都继承了 Visual 类型（无视3D部分）而在此类型里面，将会在被 VisualBrush 使用的时候，调用 AddRefOnChannelForCyclicBrush 方法

```csharp
        internal virtual void AddRefOnChannelForCyclicBrush(
            ICyclicBrush cyclicBrush,
            DUCE.Channel channel)
        {
            // 忽略代码

            Dictionary<ICyclicBrush, int> cyclicBrushToChannelsMap =
                CyclicBrushToChannelsMapField.GetValue(this);

            if (cyclicBrushToChannelsMap == null)
            {
                cyclicBrushToChannelsMap = new Dictionary<ICyclicBrush, int>();
                CyclicBrushToChannelsMapField.SetValue(this, cyclicBrushToChannelsMap);
            }

            if (!cyclicBrushToChannelsMap.ContainsKey(cyclicBrush))
            {
                cyclicBrushToChannelsMap[cyclicBrush] = 1;
            }
            else
            {
                Debug.Assert(cyclicBrushToChannelsMap[cyclicBrush] > 0);

                cyclicBrushToChannelsMap[cyclicBrush] += 1;
            }

            //
            // Render the brush's visual.
            //

            cyclicBrush.RenderForCyclicBrush(channel, false);
        }
```

上面的 ICyclicBrush 定义如下

```csharp
    internal interface ICyclicBrush
    {
        void FireOnChanged();

        void RenderForCyclicBrush(DUCE.Channel channel, bool skipChannelCheck);
    }
```

所有 VisualBrush 继承了这个接口

```csharp
  public sealed partial class VisualBrush : TileBrush, ICyclicBrush
  {

  }
```

在设置 VisualBrush 的 Visual 属性的时候，将会触发 VisualPropertyChanged 函数

```csharp
        static VisualBrush()
        {
            // Initializations
            Type typeofThis = typeof(VisualBrush);
            VisualProperty =
                  RegisterProperty("Visual",
                                   typeof(Visual),
                                   typeofThis,
                                   null,
                                   new PropertyChangedCallback(VisualPropertyChanged),
                                   null,
                                   /* isIndependentlyAnimated  = */ false,
                                   /* coerceValueCallback */ null);
        }

        private static void VisualPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {

        }
```

在这个函数里面将会调用 VisualBrush 的 AddRefResource 方法

```csharp
    public sealed partial class VisualBrush : TileBrush, ICyclicBrush
    {
        private static void VisualPropertyChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
        {
        	// 忽略代码
            System.Windows.Threading.Dispatcher dispatcher = target.Dispatcher;

            if (dispatcher != null)
            {
                DUCE.IResource targetResource = (DUCE.IResource)target;
                using (CompositionEngineLock.Acquire())
                {
                    int channelCount = targetResource.GetChannelCount();

                    for (int channelIndex = 0; channelIndex < channelCount; channelIndex++)
                    {
                        DUCE.Channel channel = targetResource.GetChannel(channelIndex);
                        Debug.Assert(!channel.IsOutOfBandChannel);
                        Debug.Assert(!targetResource.GetHandle(channel).IsNull);
                        target.ReleaseResource(oldV,channel);
                        target.AddRefResource(newV,channel);
                    }
                }
            }

            target.PropertyChanged(VisualProperty);
        }
    }
```

在 AddRefResource 函数里面将会调用上文的具体的 Visual 的 AddRefOnChannelForCyclicBrush 方法

```csharp
    public sealed partial class VisualBrush : TileBrush, ICyclicBrush
    {
        internal void AddRefResource(Visual visual, DUCE.Channel channel)
        {
            if (visual != null)
            {
                visual.AddRefOnChannelForCyclicBrush(this, channel);
            }
        }
    }
```

因此在 Visual 里面是可以了解到当前的的对象被哪些 VisualBrush 捕获

而在 Visual 里面存放的字典是不开放的，需要使用本文的反射的方式才能拿到对象从而了解这个控件是否作为 VisualBrush 的内容

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/04a3f64cc878d8e4890e72877ff08e473b4fc1a8/CalbuhewaNallrolayrani) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/04a3f64cc878d8e4890e72877ff08e473b4fc1a8/CalbuhewaNallrolayrani) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 04a3f64cc878d8e4890e72877ff08e473b4fc1a8
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 CalbuhewaNallrolayrani 文件夹


参考：[c# - How to know whether my control be used in VisualBrush - Stack Overflow](https://stackoverflow.com/q/68887185/6116637 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。