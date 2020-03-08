# dotnet Framework 源代码 · ScrollViewer

本文是分析 .net Framework 源代码的系列，主要告诉大家微软做 ScrollViewer 的思路，分析很简单。

看完本文，可以学会如何写一个 ScrollViewer ，如何定义一个 IScrollInfo 或者给他滚动添加动画

<!--more-->
<!-- CreateTime:2019/10/7 13:15:25 -->


<!-- 标签：C#，.net Framework，源代码分析，wpf，ScrollViewer ，dotnet -->

<div id="toc"></div>

## 使用

下面告诉大家如何简单使用 ScrollViewer ，一般在需要滚动的控件外面放一个 ScrollViewer 就可以实现滚动。

```csharp
  <ScrollViewer HorizontalScrollBarVisibility="Auto">
    <StackPanel VerticalAlignment="Top" HorizontalAlignment="Left">
      <TextBlock TextWrapping="Wrap" Margin="0,0,0,20">Scrolling is enabled when it is necessary. 
      Resize the window, making it larger and smaller.</TextBlock>
      <Rectangle Fill="Red" Width="500" Height="500"></Rectangle>
    </StackPanel>
  </ScrollViewer>
```

但不是所有的控件外面放一个 ScrollViewer 都能实现滚动，因为滚动实际上需要控件自己做。

## 原理

下面来告诉大家滚动是如何做的。

一个最简单的方法是设置元素的 `transForm.Y` 通过这个方式进行滚动是最简单的方法，但是缺点是其他控件不能做其他的移动。

在 ScrollViewer 存在两个滚动方式，物理滚动 和 逻辑滚动，如果使用 物理滚动 那么滚动就是ScrollViewer做的，如何使用逻辑滚动，那么滚动就是控件自己做的。

那么我从 ScrollViewer 接收输入开始讲起

### 输入

如果大家使用 ScrollViewer 进行滚动，那么也许会遇到一个神奇的需求，如何在触摸下滚动。是的，如果使用一个简单的 ScrollViewer 是无法使用触摸滚动

请看代码，写一个简单的 ScrollViewer 里面有一些矩形，可以看到这时可以进行鼠标滚动，但是触摸是无法滚动。

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F2018313153148.jpg)

```csharp
    <Grid>
        <ScrollViewer>
            <StackPanel x:Name="HcrkKmqnnfzo"></StackPanel>
        </ScrollViewer>
    </Grid>
```

在后台遍历颜色然后添加

```csharp
        public MainWindow()
        {
            InitializeComponent();

            foreach (var temp in typeof(Brushes)
                .GetProperties(BindingFlags.Static | BindingFlags.Public | BindingFlags.NonPublic)
                .Select(temp => temp.GetValue(null, null)))
            {
                var rectangle = new Rectangle
                {
                    Height = 20,
                    Fill = (Brush)temp
                };

                HcrkKmqnnfzo.Children.Add(rectangle);
            }
        }
```

代码：[WPF ScrollView 代码解释 1.1-CSDN下载](https://download.csdn.net/download/lindexi_gd/10284083 )

如果没有csdn积分，尝试使用 [我的网盘](http://lindexi.ml:8080/index.php/s/zavJRGtNeBbu8Yg )，但是我的网盘如果过期请告诉我

如果需要在触摸使用滚动，那么需要设置`PanningMode`，可以设置支持垂直拖动。

如果这时设置了`PanningMode`，就会发现拖动时让窗口抖动，这时需要在窗口重写 OnManipulationBoundaryFeedback ，请看下面代码。函数里面什么都不要写，详细请看 https://stackoverflow.com/a/6918131/6116637

```csharp
       protected override void OnManipulationBoundaryFeedback(ManipulationBoundaryFeedbackEventArgs e)
        {
        }
```

修改后的代码：[WPF ScrollView 代码解释 1.2-CSDN下载](https://download.csdn.net/download/lindexi_gd/10284122 )

那么在鼠标滚动是如何收到滚动？

从微软源代码可以看到 ScrollViewer 继承 ContentControl，所以可以重写 OnMouseWheel ，请看他的代码

```csharp
      protected override void OnMouseWheel(MouseWheelEventArgs e)
        {
            if (e.Handled) { return; }
 
            if (!HandlesMouseWheelScrolling)
            {
                return;
            }
 
            if (ScrollInfo != null)
            {
                if (e.Delta < 0) { ScrollInfo.MouseWheelDown(); }
                else { ScrollInfo.MouseWheelUp(); }
            }
 
            e.Handled = true;
        }
```

实际上 ScrollViewer 是不做滚动的，实际的滚动是 ScrollInfo 进行滚动。

### ScrollInfo

那么 ScrollInfo 是什么，实际上他是一个接口，在 ScrollViewer 里面放的控件实际上不是直接放在 ScrollViewer 里，控件是放在 `ScrollContentPresenter`，而 ScrollContentPresenter 是写在 ScrollViewer 的 Style 里，在 ScrollViewer 可以看到这个代码

```csharp
[TemplatePart(Name = "PART_ScrollContentPresenter", Type = typeof(ScrollContentPresenter))]
```

但是从垃圾微软的代码可以看到，没有属性直接使用这个，而是在使用的地方这样写`GetTemplateChild(ScrollContentPresenterTemplateName) as ScrollContentPresenter;`

这样写的性能是比较差的。

那么他是如何给 ScrollInfo 赋值？实际上在这个类的 HookupScrollingComponents 就是给 ScrollInfo 赋值，在 HookupScrollingComponents 调用的地方就是  OnApplyTemplate 所以大家可以看到，在初始化的时候就已经知道了控件。

从垃圾微软的源代码可以看到 HookupScrollingComponents 的逻辑，首先是判断属性`CanContentScroll` 判断元素里的控件是否可以滚动，如果元素里的控件可以滚动，那么再判断元素里的控件是不是继承`IScrollInfo`如果是的话，嗯，没了，就把 ScrollInfo 赋值。如果里面的控件不是继承`IScrollInfo`，那么判断一下他是不是处于列表，如果是的话就拿列表`ItemsPresenter`作为ScrollInfo。如果还是拿不到，只好用自己作为`ScrollInfo`

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F2018313162030.jpg)

从这里可以看到 CanContentScroll 如果没有设置，就直接使用这个类，也就是物理滚动就是这个类做的。如果一个元素不在列表内，不继承 IScrollInfo 那么即使设置使用逻辑滚动，实际上也是物理滚动。物理滚动就是元素不知道滚动，所有的移动都是元素无法控制。和物理滚动不同，逻辑的就是元素控制所有滚动。

### 物理滚动

下面来告诉大家，物理滚动是如何做，实际上的滚动就是在布局中使用下面的代码，让元素布局在滚动的地方，所以看起来就是元素滚动

```csharp
                  Rect childRect = new Rect(child.DesiredSize);
 
                        if (IsScrollClient)
                        {
                            childRect.X = -HorizontalOffset;
                            childRect.Y = -VerticalOffset;
                        }
 
                        //this is needed to stretch the child to arrange space,
                        childRect.Width = Math.Max(childRect.Width, arrangeSize.Width);
                        childRect.Height = Math.Max(childRect.Height, arrangeSize.Height);
 
                        child.Arrange(childRect);
```

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F2018313201221.jpg)

可以看到布局设置反过来的 HorizontalOffset 作为元素的 x 移动，通过这样就可以让元素移动

但是元素如果移动在 ScrollViewer 外面，如何裁剪？实际上就是使用重写了 GetLayoutClip 进行裁剪

```csharp
 return new RectangleGeometry(new Rect(RenderSize));
```

从代码可以知道，实际上的 ScrollViewer 是不会滚动元素的，滚动元素的是 ScrollViewer 里面的元素，滚动的方式一般都使用在布局的时候设置元素的 X、Y 来让元素滚动。我看了 StackPanel 和其他几个类，都是使用这个方式，因为对比 Translate 的方式，这个方法不会用到 Translate 也就不会在用户修改 Translate 的时候无法移动。另外这个方法是在布局做的，直接计算，如果修改 Translate 还需要在布局重新计算，所以这个方法的性能会比较高。

### 触摸输入

那么 ScrollViewer 是如何在触摸的时候获得输入？实际上在触摸的时候用的是 Manipulation ，在判断 PanningMode 给值

```csharp
                    if (panningMode == PanningMode.HorizontalOnly)
                    {
                        e.Mode = ManipulationModes.TranslateX;
                    }
                    else if (panningMode == PanningMode.VerticalOnly)
                    {
                        e.Mode = ManipulationModes.TranslateY;
                    }
                    else
                    {
                        e.Mode = ManipulationModes.Translate;
                    }
```

所以在 ManipulationDelta 可以拿到移动的值，因为直接拿到的值就是用户希望的路径所以直接设置不需要计算

但是需要倍数 PanningRatio ，如果需要惯性，那么只需要设置惯性就可以。

大概整个源代码只有这些，很多的代码都是在判断边界，还有处理一些用户输入。

在触摸的时候，核心的代码是 ManipulateScroll ，传入了当前的移动和累计的移动、是否水平移动。通过判断当前的移动是否有移动然后乘以倍数，然后通过设置 HorizontalOffset 这几个属性的值，重新布局就可以。

所以所有的代码实际上就是获得输入，然后传入给对应的 ScrollInfo ，通过 ScrollInfo 实现的方法做具体的业务。

不过 ScrollViewer 不是直接传入 ScrollInfo 需要移动的，而且发送命令

```csharp
     
        public void ScrollToHorizontalOffset(double offset)
        {
            double validatedOffset = ScrollContentPresenter.ValidateInputOffset(offset, "offset");
 
            // Queue up the scroll command, which tells the content to scroll.
            // Will lead to an update of all offsets (both live and deferred).
            EnqueueCommand(Commands.SetHorizontalOffset, validatedOffset, null);
        }
 
```

然后在具体的函数 ExecuteNextCommand 拿出一个个的命令，进行移动

```csharp
     private bool ExecuteNextCommand()
        {
            IScrollInfo isi = ScrollInfo;
 
            Command cmd = _queue.Fetch();
            switch(cmd.Code)
            {
                case Commands.LineUp:    isi.LineUp();    break;
                case Commands.LineDown:  isi.LineDown();  break;
                case Commands.LineLeft:  isi.LineLeft();  break;
                case Commands.LineRight: isi.LineRight(); break;
                //去掉差不多的代码
                case Commands.Invalid: return false;
            }
            return true;
        }
```

在输入的时候可能输入太快，而布局不是立刻进行布局，从代码可以看到，移动的业务就是在布局修改值，但是布局修改不是优先级很高的，但是输入的优先级是很高的，可能在布局的过程就不停输入。所以就需要把输入的命令放入，使用一个函数一个个拿出来，对不同的命令处理，最后再布局。

参见：

[在WPF中实现平滑滚动 - 天方 - 博客园](http://www.cnblogs.com/TianFang/p/4198731.html )

[IScrollInfo in Avalon part I – BenCon's WebLog](https://blogs.msdn.microsoft.com/bencon/2006/01/06/iscrollinfo-in-avalon-part-i/ )

[IScrollInfo in Avalon part II – BenCon's WebLog](https://blogs.msdn.microsoft.com/bencon/2006/01/07/iscrollinfo-in-avalon-part-ii/ )

[IScrollInfo in Avalon part III – BenCon's WebLog](https://blogs.msdn.microsoft.com/bencon/2006/01/08/iscrollinfo-in-avalon-part-iii/ )

[IScrollInfo tutorial part IV – BenCon's WebLog](https://blogs.msdn.microsoft.com/bencon/2006/12/09/iscrollinfo-tutorial-part-iv/ )

## 其他源代码分析

[.net Framework 源代码 · ScrollViewer](https://lindexi.gitee.io/post/.net-Framework-%E6%BA%90%E4%BB%A3%E7%A0%81-ScrollViewer.html )

[.net源码分析 – List<T> - 布鲁克石 - 博客园](http://www.cnblogs.com/brookshi/p/5353021.html )
    
[一站式WPF--依赖属性（DependencyProperty）一 - 周永恒 - 博客园](http://www.cnblogs.com/Zhouyongh/archive/2009/09/10/1564099.html )


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
