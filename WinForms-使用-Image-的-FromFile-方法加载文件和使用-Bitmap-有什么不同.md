
# WinForms 使用 Image 的 FromFile 方法加载文件和使用 Bitmap 有什么不同

本文来告诉大家使用 GDI+ 的 Image.FromFile 加载图片文件和使用创建 Bitmap 传入图片文件有什么不同

<!--more-->


<!-- 发布 -->

如使用下面代码加载图片

```csharp
                using var image = Image.FromFile(imageFile, true);
                using var bitmap = new Bitmap(image);
```

和使用下面代码加载图片

```csharp
using var bitmap = new Bitmap(imageFile);
```

不同在于使用 Image.FromFile 加载图片文件，将会进入默认解码模式，拿到的 bitmap 的格式是 32 位色的，相当于如下代码

```csharp
                var image = bitmap.Clone(new Rectangle(0, 0, cols, rows), PixelFormat.Format32bppArgb);
```

而如果是从 Bitmap 创建传入图片文件，那么图片的 PixelFormat 就是图片文件自己定义的





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。