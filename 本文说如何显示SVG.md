
# 本文说如何显示SVG

【】
本来在我一个白天晚上按钮，使用图片，图片不清晰
![这里写图片描述](http://img.blog.csdn.net/20160729180053912)
这些图片在http://www.zcool.com.cn/，不知道是不是不能直接用
我们需要一个看起来不会模糊，因为矢量图，所以我们就使用svg，其实png也是，但是他播放模糊。

<!--more-->



<div id="toc"></div>
<!-- csdn -->

lindexi

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- Generator: Adobe Illustrator 16.0.0, SVG Export Plug-In . SVG Version: 6.00 Build 0) -->
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
 width="64px" height="64px" viewBox="0 0 64 64" enable-background="new 0 0 64 64" xml:space="preserve">
<g>
 <circle fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" cx="32" cy="32" r="16"/>
 <line fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" x1="32" y1="10" x2="32" y2="0"/>
 <line fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" x1="32" y1="64" x2="32" y2="54"/>
 <line fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" x1="54" y1="32" x2="64" y2="32"/>
 <line fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" x1="0" y1="32" x2="10" y2="32"/>
 <line fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" x1="48" y1="16" x2="53" y2="11"/>
 <line fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" x1="11" y1="53" x2="16" y2="48"/>
 <line fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" x1="48" y1="48" x2="53" y2="53"/>
 <line fill="none" stroke="#000000" stroke-width="2" stroke-miterlimit="10" x1="11" y1="11" x2="16" y2="16"/>
</g>
</svg>

```

我们开始使用Image，但是没有显示

于是网上有一个库Mntone.SvgForXaml，https://github.com/mntone/SvgForXaml，我们用Nuget

![这里写图片描述](http://img.blog.csdn.net/20160729185702468)

安装Mntone.SvgForXaml，安装win2d 1.11.0

我们上面那个代码就是svg，我们使用ViewModel绑定，绑定内容是`SvgDocument`

自然我们需要写一个字符串去转换

```csharp
 private void Svgimage()
 {
 string str = @"<?xml version=""1.0"" encoding=""utf-8""?>
<!-- Generator: Adobe Illustrator 16.0.0, SVG Export Plug-In . SVG Version: 6.00 Build 0) -->
<!DOCTYPE svg PUBLIC ""-//W3C//DTD SVG 1.1//EN"" ""http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd"">
<svg version=""1.1"" id=""Layer_1"" xmlns=""http://www.w3.org/2000/svg"" xmlns:xlink=""http://www.w3.org/1999/xlink"" x=""0px"" y=""0px""
 width=""64px"" height=""64px"" viewBox=""0 0 64 64"" enable-background=""new 0 0 64 64"" xml:space=""preserve"">
<g>
 <circle fill=""none"" stroke=""#000000"" stroke-width=""2"" stroke-miterlimit=""10"" cx=""32"" cy=""32"" r=""16""/>
 <line fill=""none"" stroke=""#000000"" stroke-width=""2"" stroke-miterlimit=""10"" x1=""32"" y1=""10"" x2=""32"" y2=""0""/>
 <line fill=""none"" stroke=""#000000"" stroke-width=""2"" stroke-miterlimit=""10"" x1=""32"" y1=""64"" x2=""32"" y2=""54""/>
 <line fill=""none"" stroke=""#000000"" stroke-width=""2"" stroke-miterlimit=""10"" x1=""54"" y1=""32"" x2=""64"" y2=""32""/>
 <line fill=""none"" stroke=""#000000"" stroke-width=""2"" stroke-miterlimit=""10"" x1=""0"" y1=""32"" x2=""10"" y2=""32""/>
 <line fill=""none"" stroke=""#000000"" stroke-width=""2"" stroke-miterlimit=""10"" x1=""48"" y1=""16"" x2=""53"" y2=""11""/>
 <line fill=""none"" stroke=""#000000"" stroke-width=""2"" stroke-miterlimit=""10"" x1=""11"" y1=""53"" x2=""16"" y2=""48""/>
 <line fill=""none"" stroke=""#000000"" stroke-width=""2"" stroke-miterlimit=""10"" x1=""48"" y1=""48"" x2=""53"" y2=""53""/>
 <line fill=""none"" stroke=""#000000"" stroke-width=""2"" stroke-miterlimit=""10"" x1=""11"" y1=""11"" x2=""16"" y2=""16""/>
</g>
</svg>
";
 Svg=SvgDocument.Parse(str);

 }
```

然后我们在我们的界面

先使用命名`Mntone.SvgForXaml.UI.Xaml`

```csharp
xmlns:svg="using:Mntone.SvgForXaml.UI.Xaml"
```

然后绑定

```xml
 <Grid>
 <svg:SvgImage x:Name="Svg" Content="{x:Bind View.Svg,Mode=OneWay}"></svg:SvgImage>
 </Grid>
```

当然我们也可以放在我们的解决方案，假如我们的图片 Assets/weather_sun.svg

那么我们可以给我们的svgImage一个`x:Name="Svg"`

```csharp
file = await StorageFile.GetFileFromApplicationUriAsync(new Uri("ms-appx:///Assets/weather_sun.svg"));

await Svg.LoadFileAsync(file);
```

原文：因为他会占用内存，我们需要手动把它释放

我们写在我们页面关掉，其实这个并不是关掉，只是我们的页面不显示

```csharp
protected override void OnNavigatingFrom(NavigatingCancelEventArgs e)
{
  Svg.SafeUnload();
}
```



我们可以简单把svg转换为我们之前的图片，JPG，png

先让用户选择保存的文件，然后选择.jpg或.png

```csharp
var picker = new FileSavePicker();
 picker.SuggestedStartLocation = PickerLocationId.PicturesLibrary;
 picker.DefaultFileExtension = ".png";
 picker.FileTypeChoices.Add(new KeyValuePair<string, IList<string>>("Bitmap image", new[] { ".bmp" }.ToList()));
 picker.FileTypeChoices.Add(new KeyValuePair<string, IList<string>>("Png image", new[] { ".png" }.ToList()));
 picker.FileTypeChoices.Add(new KeyValuePair<string, IList<string>>("Jpeg image", new[] { ".jpg", ".jpe", ".jpeg" }.ToList()));
 picker.FileTypeChoices.Add(new KeyValuePair<string, IList<string>>("Gif image", new[] { ".gif" }.ToList()));

```

`SvgImageRendererFileFormat format;`可以`SvgImageRendererFileFormat.Bitmap`或者什么自己选

```csharp
await SvgImageRenderer.RendererImageAsync(file, new SvgImageRendererSettings()
{
 Document = content,
 Format = format,
 Scaling = 0.1f,
 Quality = 0.95f
});
```

![这里写图片描述](http://img.blog.csdn.net/20160731160141698)


参见：http://igrali.com/2015/12/24/how-to-render-svg-in-xaml-windows-10-uwp/

代码：https://github.com/lindexi/UWP/tree/master/uwp/src/ScalableVectorGraphic

因为已经有人写了，和我一样，驸马说他用过Svg to Xaml 做，我求请教了驸马大神，在github找到库，好像简单。






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。