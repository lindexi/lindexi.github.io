# C＃ 枚举转字符串

有时候需要把枚举转字符串，那么如何把枚举转字符串？

<div id="toc"></div>
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->

## 枚举转字符串

假如需要把枚举转字符串，可以直接把他进行转换，请看代码

```csharp
        public enum Di
        {
            /// <summary>
            /// 轨道
            /// </summary>
            Railway,

            /// <summary>
            /// 河流
            /// </summary>
            River,
        }

        static void Main(string[] args)
        {
            Console.WriteLine(Di.Railway.ToString());
        }
```

这样就可以把枚举转字符串

除了这个方法，可以使用 C# 6.0 的关键字，请看代码

```csharp
            Console.WriteLine(nameof(Di.Railway));
```

## 字符串转枚举

如果把一个枚举转字符串，那么如何把字符串转枚举？可以使用 `Enum.Parse` 不过这个方法可以会抛异常，所以使用需要知道字符串是可以转

```csharp
        public enum Di
        {
            /// <summary>
            /// 轨道
            /// </summary>
            Railway,

            /// <summary>
            /// 河流
            /// </summary>
            River,
        }

             static void Main(string[] args)
        {
            string str = Di.Railway.ToString();
            Console.WriteLine(Enum.Parse(typeof(Di), str).ToString());
        }
```

如果对于不确定的字符串，包括空的值，可以采用 TryParse 方法

```csharp
            if (Enum.TryParse(typeof(Di),null,out var value))
            {
                
            }
```

上面代码只会返回 false 不会提示无法转换

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/977b87caa80a51318e8a6e2afe77222c77f54961/BepirquwiKedoucawji)欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。