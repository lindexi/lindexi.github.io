# C# winforms 输入颜色转换颜色名

本文告诉大家如何输入颜色，如`0xFFFF8000`转换为 Orange 在 winforms 程序

<!--more-->
<!-- CreateTime:2018/9/30 18:27:49 -->


可以使用下面代码转换

```csharp
    public static class HexColorTranslator
    {
        private static Dictionary<string, string> _hex2Name;

        private static Dictionary<string, string> Hex2Name
        {
            get
            {
                if (_hex2Name == null)
                {
                    _hex2Name = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
                    foreach (KnownColor ce in typeof(KnownColor).GetEnumValues())
                    {
                        var name = ce.ToString();
                        var color = Color.FromKnownColor(ce);
                        var hex = HexConverter(color);
                        _hex2Name[hex] = name;
                    }
                }

                return _hex2Name;
            }
        }

        private static string HexConverter(Color c)
        {
            return c.R.ToString("X2") + c.G.ToString("X2") + c.B.ToString("X2");
        }

        public static string GetKnownColorFromHex(string hex)
        {
            hex = hex.TrimStart('#');
            if (Hex2Name.TryGetValue(hex, out var result))
            {
                return result;
            }

            return "???";
        }
    }

```

调用的方式是传入颜色

```csharp
HexColorTranslator.GetKnownColorFromHex("#FFFF8000");
```

参见：
https://stackoverflow.com/a/51238017/6116637

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
