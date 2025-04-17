
# WPF 从文件加载字体

本文告诉大家从文件加载字体的方法。在 wpf 使用 FontFamily 显示指定的 ttf 文件来显示字体

<!--more-->


<!-- CreateTime:2019/8/2 17:10:33 -->

假如知道了字体所在的本机的文件路径，那可以使用 PrivateFontCollection 添加字体

如下面的代码将使用本地的 `C:\Projects\MyProj\free3of9.ttf` 路径的字体，需要注意添加的 FontFamily 是需要知道字体名，和传入 PrivateFontCollection 才可以使用

```csharp
using System.Drawing;

PrivateFontCollection collection = new PrivateFontCollection();
collection.AddFontFile(@"C:\Projects\MyProj\free3of9.ttf");
FontFamily fontFamily = new FontFamily("Free 3 of 9", collection);
Font font = new Font(fontFamily, height);
```

以上方法需要引入 PrivateFontCollection 类，看起来代码比较多，另一个方法是去掉字体的后缀名，直接写在 FontFamily ，我比较希望使用下面的方法

```csharp
using System.Drawing;

FontFamily fontFamily = new FontFamily(@"C:\Projects\MyProj\#free3of9");
```

在 WPF 里面 FontFamily 存在与 System.Drawing 和 System.Windows.Media 命名空间下，同时两个命名空间的字体是不能互换的。以上方法使用的是 System.Drawing 命名空间的字体。这两个命名空间的含义和功能是完全不相同的，在 System.Drawing 命名空间下是 WinForms 的 GDI 系列的功能，在 System.Windows.Media 命名空间下是 WPF 框架内的功能。但无论是哪个，都可以在 WPF 应用下运行，请不要担心在 WPF 下引用任何 WinForms 的功能，因为引入 WinForms 对 WPF 而言是非常友好的，在 WPF 框架内部默认就引用了 WinForms 的功能

对 System.Windows.Media 命名空间的 FontFamily 类，需要使用以下两个方法之一拿到本地字体

第一个方法是通过 URI 加上字体名

```csharp
var file = @"C:\lindexi\xx.ttf";
var uri = new Uri(file);
FontFamily fontFamily = new FontFamily(uri, "字体名");
```

可以通过双击字体看到字体名，或通过下面的第二个方法拿到字体名同时使用本地字体

```csharp
            var fontFile = @"C:\lindexi\xx.ttf";

            var glyphTypeface = new GlyphTypeface(new Uri(fontFile));

            // 获取字体名
            var fontName = glyphTypeface.FamilyNames.Values.FirstOrDefault();
            var directory = Path.GetDirectoryName(fontFile);
            var fontUri = $"{new Uri(directory).AbsoluteUri}/#{fontName}";
            var fontFamily = new FontFamily(fontUri);
```

[https://stackoverflow.com/a/24022783/6116637](https://stackoverflow.com/a/24022783/6116637)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。