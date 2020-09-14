
# WPF 解决 Skia 因为找不到字体而绘制不出中文字符

在 WPF 使用 Skia 做渲染工具，如果绘制的中文都是方块，也许是字体的问题。字体的问题是 Skia 没有找到字体，本文告诉大家如何修复

<!--more-->


<!-- CreateTime:2020/8/31 12:30:28 -->

<!-- 发布 -->

在 Skia 使用特定字体，可以使用 SkiaSharp 的 SKTypeface 方法指定

```csharp
            var name = "微软雅黑";
            var skTypeface = SKTypeface.FromFamilyName(name);
```

但是如何判断 Skia 找不到字体？可以判断字体名

```csharp
            var name = "微软雅黑";
            var skTypeface = SKTypeface.FromFamilyName(name);
            if (skTypeface.FamilyName != name)
            {
                // 字体加载失败了
            }
```

解决方法是通过 WPF 辅助拿到字体，请看代码

```csharp
            var fontFamily = new FontFamily(name);
            foreach (var familyNamesValue in fontFamily.FamilyNames.Values)
            {
                skTypeface = SKTypeface.FromFamilyName(familyNamesValue);
                if (skTypeface.FamilyName == familyNamesValue)
                {
                    break;
                }
            }
```

上面代码存在的坑是 SKTypeface 是需要手动释放的资源，因此优化的代码如下

```csharp
            var name = "微软雅黑";
            var skTypeface = SKTypeface.FromFamilyName(name);
            if (skTypeface.FamilyName != name)
            {
                // 字体加载失败了
                skTypeface.Dispose();
            }

            var fontFamily = new FontFamily(name);
            foreach (var familyNamesValue in fontFamily.FamilyNames.Values)
            {
                skTypeface = SKTypeface.FromFamilyName(familyNamesValue);
                if (skTypeface.FamilyName == familyNamesValue)
                {
                    break;
                }
                else
                {
                    skTypeface.Dispose();
                }
            }
```

需要在绘制之后调用 Dispose 释放字体





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。