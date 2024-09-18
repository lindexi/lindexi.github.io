---
title: dotnet UNO 如何在调试下输出界面层级结构
description: 本文将告诉大家如何在 UNO 里面将界面的层级结构输出到调试窗口

<!--more-->

tags: dotnet
category: 
---

<!-- CreateTime:2023/11/25 9:34:35 -->

<!-- 发布 -->
<!-- 博客 -->

实现方法非常简单，和 WPF 或 UWP 等的方法是一样的，那就是通过可视化树遍历的方式，如以下代码

```csharp
    static class UISpyHelper
    {
        public static void Spy(this DependencyObject element)
        {
            Uno.Extensions.IndentedStringBuilder builder = new ();
            SpyInner(element, builder);
            var spyText = builder.ToString();
            Debug.WriteLine(spyText);
        }

        private static void SpyInner(DependencyObject element, Uno.Extensions.IndentedStringBuilder builder)
        {
            var name = string.Empty;
            if (element is FrameworkElement frameworkElement)
            {
                name = frameworkElement.Name;
            }
            builder.AppendLine($"{name}({element.GetType().FullName})\r\n");

            for (var i = 0; i < VisualTreeHelper.GetChildrenCount(element); i++)
            {
                using var t = builder.Indent();
                var child = VisualTreeHelper.GetChild(element, i);
                SpyInner(child, builder);
            }
        }
    }
```

通过以上的代码即可将传入的 UI 上的控件进行遍历输出层次结构。比如传入一个 TextBox 控件，可以看到大概如下的输出内容

```
TextBox(Microsoft.UI.Xaml.Controls.TextBox)
  RootBorder(Microsoft.UI.Xaml.Controls.Border)
      Root(Microsoft.UI.Xaml.Controls.Grid)
          (Microsoft.UI.Xaml.Controls.Border)
              IconPresenter(Microsoft.UI.Xaml.Controls.ContentPresenter)
              ContentElement(Microsoft.UI.Xaml.Controls.ScrollViewer)
              (Microsoft.UI.Xaml.Controls.Border)
                  Root(Microsoft.UI.Xaml.Controls.Grid)
                      ScrollContentPresenter(Microsoft.UI.Xaml.Controls.ScrollContentPresenter)
                          (Microsoft.UI.Xaml.Controls.TextBlock)
                          VerticalScrollBar(Microsoft.UI.Xaml.ElementStub)
                        HorizontalScrollBar(Microsoft.UI.Xaml.ElementStub)
                        ScrollBarSeparator(Microsoft.UI.Xaml.Controls.Border)
                  (Microsoft.UI.Xaml.Controls.Border)
              PlaceholderElement(Microsoft.UI.Xaml.Controls.TextBlock)
              DeleteButton(Microsoft.UI.Xaml.Controls.Button)
              ButtonLayoutGrid(Microsoft.UI.Xaml.Controls.Grid)
                  GlyphElement(Microsoft.UI.Xaml.Shapes.Path)
```

在 UNO 里面也有自带的方法，通过 TreeGraph 打印出来，代码如下

```csharp
using Uno.UI.Extensions;

        var treeGraph = this.TreeGraph();
```

以上的 `treeGraph` 在我的简单的应用的输出内容大概如下

```
ConnectionUserControl // Actual=624x379, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
    Grid // Actual=624x379, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
        Grid // Actual=624x48, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
            Image // Actual=24x24, HV=Left/Stretch, Margin=[18,0,0,0], Opacity=1, Visibility=Visible
            TextBlock // Actual=146x20.3203125, HV=Stretch/Center, Margin=[50,0,0,0], Padding=0, Opacity=1, Visibility=Visible
            Button#AboutButton // Actual=24x24, HV=Right/Center, CornerRadius=0, Margin=[0,0,149,0], Padding=[8,4], Opacity=1, Visibility=Visible
                Grid // Actual=24x24, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
                    Path // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=1, Visibility=Visible
            Button // Actual=24x24, HV=Right/Center, CornerRadius=0, Margin=[0,0,109,0], Padding=[8,4], Opacity=1, Visibility=Visible
                Grid // Actual=24x24, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
                    Path // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=1, Visibility=Visible
            Border // Actual=1x14, HV=Right/Stretch, CornerRadius=0, Margin=[0,0,93,0], Padding=0, Opacity=1, Visibility=Visible
            Button // Actual=24x24, HV=Right/Center, CornerRadius=0, Margin=[0,0,58,0], Padding=[8,4], Opacity=1, Visibility=Visible
                Grid // Actual=24x24, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
                    Path // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=1, Visibility=Visible
            Button // Actual=24x24, HV=Right/Center, CornerRadius=0, Margin=[0,-2,18,0], Padding=[8,4], Opacity=1, Visibility=Visible
                Grid // Actual=24x24, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
                    Path // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=1, Visibility=Visible
        Grid#ContentGrid // Actual=640x371, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
            Border // Actual=360x292, HV=Stretch/Top, CornerRadius=[4,4,0,0], Margin=0, Padding=0, Opacity=1, Visibility=Visible
                Grid // Actual=360x292, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
                    TextBlock // Actual=193.546875x54.48046875, HV=Center/Stretch, Margin=[0,24,0,0], Padding=0, Opacity=1, Visibility=Visible
                    Grid // Actual=160x160, HV=Stretch/Top, CornerRadius=0, Margin=[0,68,0,0], Padding=0, Opacity=1, Visibility=Visible
                        Image // Actual=0x0, HV=Stretch/Stretch, Margin=4, Opacity=1, Visibility=Collapsed
                        Grid // Actual=0x0, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Collapsed
                            Path#ErrorPath // Actual=0x0, HV=Center/Center, Margin=0, Opacity=1, Visibility=Collapsed
                        CircleLoading#CircleLoading // Actual=32x32, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
                            Viewbox // Actual=32x32, HV=Center/Center, Margin=0, Opacity=1, Visibility=Visible
                                Border // Actual=24x24, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
                                    Grid#RootGrid // Actual=24x24, HV=Stretch/Stretch, CornerRadius=0, Margin=0, Padding=0, Opacity=1, Visibility=Visible
                                        Path#Part1 // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=0.16, Visibility=Visible
                                        Path#Part2 // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=0.28, Visibility=Visible
                                        Path#Part3 // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=0.4, Visibility=Visible
                                        Path#Part4 // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=0.52, Visibility=Visible
                                        Path#Part5 // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=0.64, Visibility=Visible
                                        Path#Part6 // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=0.76, Visibility=Visible
                                        Path#Part7 // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=0.88, Visibility=Visible
                                        Path#Part8 // Actual=24x24, HV=Stretch/Stretch, Margin=0, Opacity=1, Visibility=Visible
                        Image#QrCodeImage // Actual=160x160, HV=Stretch/Stretch, Margin=0, Opacity=1, Visibility=Visible
```

官方的 TreeGraph 方法和前文的使用 VisualTreeHelper 获取到的元素对象都是相同的，原因是 TreeGraph 方法底层实现如下

```csharp
using _View = Windows.UI.Xaml.DependencyObject;

  internal static IEnumerable<_View> EnumerateChildren(this _View? reference)
  {
    return Enumerable
      .Range(0, VisualTreeHelper.GetChildrenCount(reference))
      .Select(x => VisualTreeHelper.GetChild(reference, x));
  }
```

更多请看 [Inspecting the runtime visual tree of an Uno app](https://platform.uno/docs/articles/uno-development/debugging-inspect-visual-tree.html )
