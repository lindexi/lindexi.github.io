本文将告诉大家如何在 UNO 里面将界面的层级结构输出到调试窗口

<!--more-->


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
