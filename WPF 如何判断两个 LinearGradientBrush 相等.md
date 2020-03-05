# WPF 如何判断两个 LinearGradientBrush 相等

在 WPF 没有提供默认的判断 LinearGradientBrush 相等的方法，本文给大家一个可以直接在项目使用的方法

<!--more-->
<!-- CreateTime:2019/7/22 21:26:22 -->

<!-- csdn -->

可以使用下面代码判断两个 LinearGradientBrush 是否相等

```csharp
        public static bool AreEquals(LinearGradientBrush linearGradientBrush1,
            LinearGradientBrush linearGradientBrush2)
        {
            if (linearGradientBrush1.ColorInterpolationMode !=
                linearGradientBrush2.ColorInterpolationMode
                || linearGradientBrush1.EndPoint !=
                linearGradientBrush2.EndPoint
                || linearGradientBrush1.MappingMode !=
                linearGradientBrush2.MappingMode
                // ReSharper disable once CompareOfFloatsByEqualityOperator
                || linearGradientBrush1.Opacity !=
                linearGradientBrush2.Opacity
                || linearGradientBrush1.StartPoint !=
                linearGradientBrush2.StartPoint
                || linearGradientBrush1.SpreadMethod !=
                linearGradientBrush2.SpreadMethod
                || linearGradientBrush1.GradientStops.Count !=
                linearGradientBrush2.GradientStops.Count)
            {
                return false;
            }

            for (int i = 0; i < linearGradientBrush1.GradientStops.Count; i++)
            {
                if (linearGradientBrush1.GradientStops[i].Color !=
                    linearGradientBrush2.GradientStops[i].Color
                    // ReSharper disable once CompareOfFloatsByEqualityOperator
                    || linearGradientBrush1.GradientStops[i].Offset !=
                    linearGradientBrush2.GradientStops[i].Offset)
                {
                    return false;
                }
            }

            return true;
        }

```

上面代码可以在项目使用

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
