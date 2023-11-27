
# dotnet UNO 如何在调试下输出界面层级结构

本文将告诉大家如何在 UNO 里面将界面的层级结构输出到调试窗口

<!--more-->


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




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。