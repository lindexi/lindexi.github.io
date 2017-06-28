# WPF 从文件加载字体

本文告诉大家从文件加载字体。

<!--more-->
<!-- csdn -->

假如有字体在 `C:\Projects\MyProj\free3of9.ttf` ，可以使用  PrivateFontCollection 添加字体。

下面的代码就可以使用本地的 free3of9.ttf 

```csharp
PrivateFontCollection collection = new PrivateFontCollection();
collection.AddFontFile(@"C:\Projects\MyProj\free3of9.ttf");
FontFamily fontFamily = new FontFamily("Free 3 of 9", collection);
Font font = new Font(fontFamily, height);
```

另一个方法是去掉字体的后缀名

```csharp
FontFamily fontFamily = new FontFamily(@"C:\Projects\MyProj\#free3of9");
```

https://stackoverflow.com/a/24022783/6116637


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
