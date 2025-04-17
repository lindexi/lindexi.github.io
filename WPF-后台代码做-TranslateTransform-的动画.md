
# WPF 后台代码做 TranslateTransform 的动画

本文告诉大家，在后台代码，对 TranslateTransform 做动画的方法

<!--more-->


<!-- CreateTime:2021/6/17 20:41:16 -->

<!-- 发布 -->

今天小伙伴问我一个问题，说为什么相同的代码，如果设置到按钮上，是可以让按钮的某个属性变更，但是如果设置给 TranslateTransform 的 X 或 Y 就不会有任何值变更

在 WPF 中，通过 [官方文档](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/graphics-multimedia/storyboards-overview?WT.mc_id=WD-MVP-5003260) 里面的描述，对于 Freezable 类型的对象，如 SolidColorBrush 和 RotateTransform 和 GradientStop 等类型，都是不支持直接的动画，也就是如以下代码是不能触发动画

假定有 XAML 界面如下，期望在点击按钮时，修改按钮的 TranslateTransform 做动画

```xml
  <Grid>
    <Button x:Name="Button" HorizontalAlignment="Center" VerticalAlignment="Center" Content="按钮" Click="Button_OnClick">
      <Button.RenderTransform>
        <TranslateTransform x:Name="ButtonTranslateTransform"></TranslateTransform>
      </Button.RenderTransform>
    </Button>
  </Grid>
```

如果直接对使用 Storyboard 的 SetTarget 方法给对象设置 DoubleAnimation 将会是无效的，也就是说如以下的代码做的 TranslateTransform 动画是无效的，没有反应的

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var storyboard = new Storyboard();

            var doubleAnimation = new DoubleAnimation();
            Storyboard.SetTarget(doubleAnimation, ButtonTranslateTransform);
            Storyboard.SetTargetProperty(doubleAnimation, new PropertyPath(TranslateTransform.XProperty));

            doubleAnimation.To = 100;
            doubleAnimation.Duration = new Duration(TimeSpan.FromSeconds(2));

            storyboard.Children.Add(doubleAnimation);
            storyboard.Begin();
        }
```

如果想要给 Freezable 类型的对象做动画，可以通过间接的方法，也就是通过 Freezable 类型的对象所在的元素，使用点的方式写出来具体的代码

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var storyboard = new Storyboard();

            var doubleAnimation = new DoubleAnimation();
            Storyboard.SetTarget(doubleAnimation, Button);
            Storyboard.SetTargetProperty(doubleAnimation, new PropertyPath("(UIElement.RenderTransform).(TranslateTransform.X)"));

            doubleAnimation.To = 100;
            doubleAnimation.Duration = new Duration(TimeSpan.FromSeconds(2));

            storyboard.Children.Add(doubleAnimation);
            storyboard.Begin();
        }
```

写法就是通过某个元素的某个属性加上某个类型的某个属性。如上面代码使用的是 UIElement 的 RenderTransform 属性，这个属性的值的类型是 TranslateTransform 类型，设置这个类型的 X 属性

上面的 PropertyPath 有可以换成如下方式写

```csharp
            var propertyChain = new object[]
            {
                UIElement.RenderTransformProperty,
                TranslateTransform.XProperty
            };

            Storyboard.SetTargetProperty(doubleAnimation, new PropertyPath("(0).(1)", propertyChain));
```

我更推荐使用这个写法，因为这样就不会写错命名

而如果只是为了修改 TranslateTransform 的 X 属性，最简单的写法就是通过 BeginAnimation 的方式，如下面代码

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            ButtonTranslateTransform.BeginAnimation(TranslateTransform.XProperty, new DoubleAnimation()
            {
                To = 100,
                Duration = new Duration(TimeSpan.FromSeconds(1))
            });
        }
```

以上代码可以看到很清真

这里的 Duration 其实可以通过 TimeSpan 转换，而不需要创建 Duration 对象。然而在 WPF 依然定义 Duration 类的原因是为了支持 Duration.Automatic 和 Duration.Forever 特殊的定义

如果是需要有多个属性开始做动画，不想使用 BeginAnimation 的方式，可以通过在后台代码用 SetTargetName 的方法指定，如下面代码

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var storyboard = new Storyboard();

            var doubleAnimation = new DoubleAnimation();
            Storyboard.SetTargetName(doubleAnimation, nameof(ButtonTranslateTransform));
            Storyboard.SetTargetProperty(doubleAnimation, new PropertyPath(TranslateTransform.XProperty));

            doubleAnimation.To = 100;
            doubleAnimation.Duration = new Duration(TimeSpan.FromSeconds(2));

            var storyboardName = "s" + storyboard.GetHashCode();
            // 加入到字典，让 Storyboard 和 ButtonTranslateTransform 在相同的一个 NameScope 里
            Resources.Add(storyboardName, storyboard);

            storyboard.Children.Add(doubleAnimation);
            storyboard.Begin();
        }
```

在后台代码做动画，如果使用 SetTargetName 就需要让 Storyboard 和对应的元素在相同的一个 NameScope 里，不然将会提示 System.InvalidOperationException 不存在可解析名称“xx”的适用名称领域，如下面代码

```csharp
System.InvalidOperationException:“不存在可解析名称“ButtonTranslateTransform”的适用名称领域。”
```

上面代码通过将动画加入到资源字典的方式，让动画和元素在相同的 NameScope 而让动画能找到元素。但是上面代码将会在资源字典加入一个 Storyboard 而没有释放，如果在你的实际代码，我推荐在动画完成之后，删除资源字典的动画

我特别翻了 WPF 编程宝典，发现宝典里面没有这部分知识，也就是没有告诉大家为什么直接给 TranslateTransform 的属性做动画将会失效。好在官方文档里面有说到这点

本文代码还请到 [github](https://github.com/lindexi/lindexi_gd/tree/78f63c1c076065d1891559f5af2cb29f10a39f8b/KayceefiwhearHaijanihukere) 或 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/78f63c1c076065d1891559f5af2cb29f10a39f8b/KayceefiwhearHaijanihukere) 上阅读代码

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 78f63c1c076065d1891559f5af2cb29f10a39f8b
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 KayceefiwhearHaijanihukere 文件夹

[Storyboards Overview - WPF .NET Framework](https://docs.microsoft.com/en-us/dotnet/desktop/wpf/graphics-multimedia/storyboards-overview?WT.mc_id=WD-MVP-5003260)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。