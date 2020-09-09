
# dotnet OpenXML 如何获取 schemeClr 颜色

颜色是一个大的主题，在 ECMA 376 里面用了 19 页 A4 描述了颜色，但仅是简单的描述。在 OpenXML 定义了 Scheme Color (schemeClr) 是用来表示主题的颜色，可以跟随主题的更改而更改颜色。例如我的文本设置为主题的文本颜色，那么在我更改文档主题的文本色就可以更改我的文本颜色

<!--more-->


<!-- CreateTime:2020/9/7 14:38:11 -->

<!-- 发布 -->

在 OpenXML 的颜色里面，其中 Scheme Color （a:schemeClr） 是十分强大的，可以用来作为模版发布，让用户自己一键替换主题色。也提供了给智能排版协助更换主题色的方法

在填充笔刷里，本文说的颜色是放在 SolidColorBrush 里面，也就是在 OpenXML 的 `a:solidFill` 里面的颜色，大概的文档代码请看下面

```xml
<a:solidFill>
	<a:schemeClr val="tx1">
		<a:lumMod val="65000" />
		<a:lumOff val="35000" />
	</a:schemeClr>
</a:solidFill>
```

上面代码的 `a:solidFill` 使用 `a:schemeClr` 填充，使用的值是 `val="tx1"` 而在 `a:schemeClr`  的 lumMod 和 lumOff 表示颜色转换，更多颜色转换请看 [dotnet OpenXML 颜色变换](https://blog.lindexi.com/post/dotnet-OpenXML-%E9%A2%9C%E8%89%B2%E5%8F%98%E6%8D%A2.html )

那么 `val="tx1"` 表示的颜色是什么？是否可以转 RGB 表示，其实这个值表示的是主题里面的 tx1 也就是 Text1 属性的颜色，需要再次去主题里面找到对应的颜色

假定如上是放在 Slide 页面里面的某个文本的颜色，代码如下

```xml
<p:sp>
  <p:nvSpPr>
    <p:cNvPr id="2" name="标题 1" />
    <p:cNvSpPr>
      <a:spLocks noGrp="1" />
    </p:cNvSpPr>
    <p:nvPr>
      <p:ph type="ctrTitle" />
    </p:nvPr>
  </p:nvSpPr>
  <p:spPr>
    <a:xfrm>
      <a:off x="568575" y="84959" />
      <a:ext cx="9144000" cy="635000" />
    </a:xfrm>
  </p:spPr>
  <p:txBody>
    <a:bodyPr />
    <a:lstStyle />
    <a:p>
      <a:r>
        <a:rPr kumimoji="1" lang="zh-CN" altLang="en-US" b="1" dirty="0">
          <a:solidFill>
            <a:schemeClr val="tx1">
              <a:lumMod val="65000" />
              <a:lumOff val="35000" />
            </a:schemeClr>
          </a:solidFill>
          <a:ea typeface="微软雅黑" panose="020B0503020204020204" charset="-122" />
        </a:rPr>
        <a:t>林德熙</a:t>
      </a:r>
    </a:p>
  </p:txBody>
</p:sp>
```

此时想要拿到这个文本的字体的颜色，就需要先获取 Color Map 颜色表，然后找到 Color Scheme 读取实际颜色。默认的 Color Map 在 Slide Master 里面，关于 Slide Master 请看[dotnet OpenXML 的 Slide Master 和 Slide Layout 是什么](https://blog.lindexi.com/post/dotnet-OpenXML-%E7%9A%84-Slide-Master-%E5%92%8C-Slide-Layout-%E6%98%AF%E4%BB%80%E4%B9%88.html )

但是此时在页面里面依然可以通过 ColorMapOverride 重写颜色表，因此在 OpenXML SDK 里面需要这样获取，在拿到 SlidePart, SlideLayoutPart, SlideMasterPart 三个变量，然后先判断当前页面是否有重写，有的话使用当前页面，然后再使用 SlideLayout 的

当然，如果此时的元素是放在 Slide Layout 的元素，那么就不能使用 Slide 的，大概代码如下

```csharp
            var masterColorMap = slideMasterPart?.SlideMaster.ColorMap;

            //从当前Slide获取ColorMap
            if (slidePart?.Slide.ColorMapOverride != null)
            {
                if (slidePart.Slide.ColorMapOverride.MasterColorMapping != null)
                {
                    return masterColorMap;
                }

                if (slidePart.Slide.ColorMapOverride.OverrideColorMapping != null)
                {
                    return slidePart.Slide.ColorMapOverride.OverrideColorMapping.ToColorMap();
                }
            }

            //从SlideLayout获取ColorMap
            if (slideLayoutPart?.SlideLayout.ColorMapOverride != null)
            {
                if (slideLayoutPart.SlideLayout.ColorMapOverride.MasterColorMapping != null)
                {
                    return masterColorMap;
                }

                if (slideLayoutPart.SlideLayout.ColorMapOverride.OverrideColorMapping != null)
                {
                    return slideLayoutPart.SlideLayout.ColorMapOverride.OverrideColorMapping.ToColorMap();
                }
            }

            //从SlideMaster获取ColorMap
            return masterColorMap;
```

上面代码的 ToColorMap 是一个扩展方法，请看下面代码

```csharp
        /// <summary>
        /// 将<see cref="OverrideColorMapping"/>转换为<see cref="ColorMap"/>
        /// </summary>
        /// <param name="mapping"></param>
        /// <returns></returns>
        public static ColorMap ToColorMap(this OverrideColorMapping mapping)
        {
            return new ColorMap()
            {
                Accent1 = mapping.Accent1,
                Accent2 = mapping.Accent2,
                Accent3 = mapping.Accent3,
                Accent4 = mapping.Accent4,
                Accent5 = mapping.Accent5,
                Accent6 = mapping.Accent6,
                Background1 = mapping.Background1,
                Background2 = mapping.Background2,
                FollowedHyperlink = mapping.FollowedHyperlink,
                Hyperlink = mapping.Hyperlink,
                Text1 = mapping.Text1,
                Text2 = mapping.Text2
            };
        }
```

颜色表在 Slide Master 的内容如下

```xml
  <p:clrMap bg1="lt1" tx1="dk1" bg2="lt2" tx2="dk2" accent1="accent1" accent2="accent2" accent3="accent3" accent4="accent4" accent5="accent5" accent6="accent6" hlink="hlink" folHlink="folHlink" />
```

也就是如上面文本使用的是 tx1 的颜色，在色表可以看到 `tx1="dk1"` 所以此时使用的是 `dk1` 的颜色，这个颜色需要在主题里面找到对应的颜色

找到对应的主题的方法，在 OpenXML 里面可以使用如下方法拿到

```csharp
            //从当前Slide获取theme
            if (slidePart?.ThemeOverridePart?.ThemeOverride?.ColorScheme != null)
            {
                return slidePart.ThemeOverridePart.ThemeOverride.ColorScheme;
            }

            //从SlideLayout获取theme
            if (slideLayoutPart?.ThemeOverridePart?.ThemeOverride?.ColorScheme != null)
            {
                return slideLayoutPart.ThemeOverridePart.ThemeOverride.ColorScheme;
            }

            //从SlideMaster获取theme
            return slideMasterPart?.ThemePart?.Theme?.ThemeElements?.ColorScheme;
```

如果是放在页面的元素，那么依次尝试获取 Slide 的主题，如果拿不到，就从 SlideLayout 获取，再获取不到就从 SlideMaster 获取。如果是 Slide Layout 的元素，那么先从 SlideLayout 获取，而不能从 Slide 获取，如果获取不到就从 SlideMaster 获取

在拿到颜色表和主题，可以使用如下方法找到对应颜色

```csharp
 static Color ToColor(this SchemeColor color, ColorScheme scheme, ColorMap map)
 {
 	var value = color.Val.Value;
    value = SchemeColorMap(value, map);
    var schemeColor = FindSchemeColor(value, scheme);
    // 忽略代码
 }
```

而 ColorMap 是非必须的，如果没有 ColorMap 那么就不需要使用 SchemeColorMap 方法

```csharp
 static Color ToColor(this SchemeColor color, ColorScheme scheme, ColorMap map)
 {
 	var value = color.Val.Value;
 	if (map != null)
 	{
 		value = SchemeColorMap(value, map);
 	}
    
    var schemeColor = FindSchemeColor(value, scheme);
    // 忽略代码
 }
```

这里的 SchemeColorMap 方法代码如下

```csharp
 private static SchemeColorValues SchemeColorMap(SchemeColorValues value, ColorMap map) =>
     value switch
     {
         SchemeColorValues.Accent1 => IndexToSchemeColor(map.Accent1),
         SchemeColorValues.Accent2 => IndexToSchemeColor(map.Accent2),
         SchemeColorValues.Accent3 => IndexToSchemeColor(map.Accent3),
         SchemeColorValues.Accent4 => IndexToSchemeColor(map.Accent4),
         SchemeColorValues.Accent5 => IndexToSchemeColor(map.Accent5),
         SchemeColorValues.Accent6 => IndexToSchemeColor(map.Accent6),
         SchemeColorValues.Dark1 => SchemeColorValues.Dark1,
         SchemeColorValues.Dark2 => SchemeColorValues.Dark2,
         SchemeColorValues.FollowedHyperlink => IndexToSchemeColor(map.FollowedHyperlink),
         SchemeColorValues.Hyperlink => IndexToSchemeColor(map.Hyperlink),
         SchemeColorValues.Light1 => SchemeColorValues.Light1,
         SchemeColorValues.Light2 => SchemeColorValues.Light2,
         SchemeColorValues.Background1 => IndexToSchemeColor(map.Background1),
         SchemeColorValues.Background2 => IndexToSchemeColor(map.Background2),
         SchemeColorValues.Text1 => IndexToSchemeColor(map.Text1),
         SchemeColorValues.Text2 => IndexToSchemeColor(map.Text2),
         _ => SchemeColorValues.Accent1,
     };

 private static SchemeColorValues IndexToSchemeColor(ColorSchemeIndexValues value) =>
     value switch
     {
         ColorSchemeIndexValues.Accent1 => SchemeColorValues.Accent1,
         ColorSchemeIndexValues.Accent2 => SchemeColorValues.Accent2,
         ColorSchemeIndexValues.Accent3 => SchemeColorValues.Accent3,
         ColorSchemeIndexValues.Accent4 => SchemeColorValues.Accent4,
         ColorSchemeIndexValues.Accent5 => SchemeColorValues.Accent5,
         ColorSchemeIndexValues.Accent6 => SchemeColorValues.Accent6,
         ColorSchemeIndexValues.Dark1 => SchemeColorValues.Dark1,
         ColorSchemeIndexValues.Dark2 => SchemeColorValues.Dark2,
         ColorSchemeIndexValues.FollowedHyperlink => SchemeColorValues.FollowedHyperlink,
         ColorSchemeIndexValues.Hyperlink => SchemeColorValues.Hyperlink,
         ColorSchemeIndexValues.Light1 => SchemeColorValues.Light1,
         ColorSchemeIndexValues.Light2 => SchemeColorValues.Light2,
         _ => SchemeColorValues.Accent1,
     };
```

这里的 FindSchemeColor 代码如下

```csharp
        private static Color2Type FindSchemeColor(SchemeColorValues value, ColorScheme scheme) =>
            value switch
            {
                SchemeColorValues.Accent1 => scheme.Accent1Color,
                SchemeColorValues.Accent2 => scheme.Accent2Color,
                SchemeColorValues.Accent3 => scheme.Accent3Color,
                SchemeColorValues.Accent4 => scheme.Accent4Color,
                SchemeColorValues.Accent5 => scheme.Accent5Color,
                SchemeColorValues.Accent6 => scheme.Accent6Color,
                SchemeColorValues.Dark1 => scheme.Dark1Color,
                SchemeColorValues.Dark2 => scheme.Dark2Color,
                SchemeColorValues.FollowedHyperlink => scheme.FollowedHyperlinkColor,
                SchemeColorValues.Hyperlink => scheme.Hyperlink,
                SchemeColorValues.Light1 => scheme.Light1Color,
                SchemeColorValues.Light2 => scheme.Light2Color,
                _ => null,
            };
```

此时拿到的只是颜色，颜色可以选的值有很多

```xml
    <a:clrScheme name="Office">
      <a:dk1>
        <a:sysClr val="windowText" lastClr="000000" />
      </a:dk1>
      <a:lt1>
        <a:sysClr val="window" lastClr="FFFFFF" />
      </a:lt1>
      <a:dk2>
        <a:srgbClr val="44546A" />
      </a:dk2>
      <a:lt2>
        <a:srgbClr val="E7E6E6" />
      </a:lt2>
      <a:accent1>
        <a:srgbClr val="4472C4" />
      </a:accent1>
      <a:accent2>
        <a:srgbClr val="ED7D31" />
      </a:accent2>
      <a:accent3>
        <a:srgbClr val="A5A5A5" />
      </a:accent3>
      <a:accent4>
        <a:srgbClr val="FFC000" />
      </a:accent4>
      <a:accent5>
        <a:srgbClr val="5B9BD5" />
      </a:accent5>
      <a:accent6>
        <a:srgbClr val="70AD47" />
      </a:accent6>
      <a:hlink>
        <a:srgbClr val="0563C1" />
      </a:hlink>
      <a:folHlink>
        <a:srgbClr val="954F72" />
      </a:folHlink>
    </a:clrScheme>
```

在这个颜色里面 dk1 的颜色是 sysClr 也就是 SystemColor 颜色，读取 lastClr 也就是 LastColor 属性就可以拿到颜色。而 SystemColor 的 val 表示的是使用系统的哪个颜色，因此是可以做到在不同的系统设置打开的文档的颜色是不同的。这部分获取的逻辑就相对复杂了，也不在本文范围

本文的 Scheme Color 可以在 ECMA 376 的 20.1.2.3.29 schemeClr (Scheme Color) 找到。如果没有 OpenXML SDK 的定义辅助，也许这里的逻辑能坑你很久

更多请看 [Office 使用 OpenXML SDK 解析文档博客目录](https://blog.lindexi.com/post/Office-%E4%BD%BF%E7%94%A8-OpenXML-SDK-%E8%A7%A3%E6%9E%90%E6%96%87%E6%A1%A3%E5%8D%9A%E5%AE%A2%E7%9B%AE%E5%BD%95.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。