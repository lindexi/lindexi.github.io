# win10 uwp 颜色转换

本文告诉大家如何从字符串转颜色，从颜色转字符串

<!--more-->
<!-- csdn -->

## 字符串转颜色

在 WPF 可以使用下面的代码把十六进制的颜色字符串转颜色

```csharp
            Color color = (Color) ColorConverter.ConvertFromString("#FFDFD991");

```

```csharp
string hex = "#FFFFFF";  
Color color = System.Drawing.ColorTranslator.FromHtml(hex); 
```

但是 UWP 没这个方法，所以需要自己写一个方法

```csharp
        public SolidColorBrush GetSolidColorBrush(string hex)
        {
            hex = hex.Replace("#", string.Empty);
            byte a = (byte) (Convert.ToUInt32(hex.Substring(0, 2), 16));
            byte r = (byte) (Convert.ToUInt32(hex.Substring(2, 2), 16));
            byte g = (byte) (Convert.ToUInt32(hex.Substring(4, 2), 16));
            byte b = (byte) (Convert.ToUInt32(hex.Substring(6, 2), 16));
            return new SolidColorBrush(Windows.UI.Color.FromArgb(a, r, g, b));
        }
```

## 颜色转字符串

如果需要从颜色转字符串是很简单

```csharp
Color.ToString()
```

上面的代码就可以输出字符串

![](https://i.loli.net/2018/04/08/5aca000c4b395.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
