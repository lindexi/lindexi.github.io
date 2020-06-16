
# WPF 通过辅助方法在 csharp 代码写出 XAML 界面效果

我看到了 MUV 的写法，发现其实默认 WPF 也是支持了大部分了，小部分还不支持的需要改一下 WPF 框架，反正现在 WPF 框架也开源了，我也算是 WPF 框架的开发者，也能构建发布自己的版本

<!--more-->


<!-- 发布 -->

本文的内容不需要使用德熙发布的版本，而是默认的 WPF 就可以支持了，写出的效果如下

```csharp
            var border = new Border()
            {
                Background = Brushes.Gray,
                Width = 100,
                Height = 100,
                Child = new Grid
                {
                    Children =
                    {
                        new StackPanel()
                        {
                            Orientation = Orientation.Horizontal,
                            VerticalAlignment = VerticalAlignment.Bottom,
                            Margin = new Thickness(10, 10, 10, 10),
                            Children =
                            {
                                new Button
                                {

                                }.Do(b => { b.Click += Foo_Click; })
                            }
                        }
                    }
                }
            };
```

在 C# 写界面代码的时候会遇到的问题是事件的监听等问题，本文主要是解决事件监听的问题，写法很简单，添加下面这个辅助方法

```csharp
    public static class UIInitExtensions
    {
        public static Button Do(this Button button, Action<Button> action)
        {
            action(button);
            return button;
        }
    }
```

注意的点是扩展方法需要返回自身，这样才能在后台代码写





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。